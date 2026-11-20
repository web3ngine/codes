===== admin\services.php =====
<?php
declare(strict_types=1);

require_once __DIR__ . '/../includes/auth.php';
require_once __DIR__ . '/../config/database.php';

requireAdmin($pdo);

function servicesRedirect(string $message, string $type = 'success'): never
{
    $_SESSION['services_flash'] = [
        'message' => $message,
        'type' => $type
    ];

    header('Location: services.php');
    exit;
}

function servicesLog(PDO $pdo, string $action, string $details): void
{
    $stmt = $pdo->prepare(
        'INSERT INTO activity_logs (user_id, action, details, ip_address)
         VALUES (:user_id, :action, :details, :ip)'
    );

    $stmt->execute([
        ':user_id' => (int)$_SESSION['admin_id'],
        ':action' => $action,
        ':details' => $details,
        ':ip' => $_SERVER['REMOTE_ADDR'] ?? null
    ]);
}

function servicesPostString(string $key): string
{
    return trim((string)($_POST[$key] ?? ''));
}

function servicesEscape(mixed $value): string
{
    return htmlspecialchars((string)$value, ENT_QUOTES, 'UTF-8');
}

$allowedStatuses = ['pending', 'active', 'suspended', 'terminated'];

if ($_SERVER['REQUEST_METHOD'] === 'POST') {
    $csrf = (string)($_POST['csrf_token'] ?? '');

    if (
        empty($_SESSION['admin_csrf']) ||
        !hash_equals($_SESSION['admin_csrf'], $csrf)
    ) {
        servicesRedirect('Security token expired. Please try again.', 'error');
    }

    $action = servicesPostString('action');
    $id = filter_var($_POST['id'] ?? null, FILTER_VALIDATE_INT);

    if (!$id || $id < 1) {
        servicesRedirect('Invalid service ID.', 'error');
    }

    try {
        $stmt = $pdo->prepare(
            'SELECT id, domain, username, status
             FROM hosting_services
             WHERE id = :id
             LIMIT 1'
        );
        $stmt->execute([':id' => $id]);
        $service = $stmt->fetch();

        if (!$service) {
            throw new RuntimeException('Service not found.');
        }

        if ($action === 'update') {
            $domain = servicesPostString('domain');
            $username = servicesPostString('username');

            if (
                $domain === '' ||
                mb_strlen($domain) > 190 ||
                preg_match('/[\s\/\\\\]/', $domain)
            ) {
                throw new RuntimeException('Enter a valid domain name.');
            }

            if (
                $username === '' ||
                mb_strlen($username) > 100
            ) {
                throw new RuntimeException('Username is required and must not exceed 100 characters.');
            }

            $pdo->beginTransaction();

            $update = $pdo->prepare(
                'UPDATE hosting_services
                 SET domain = :domain, username = :username
                 WHERE id = :id'
            );

            $update->execute([
                ':domain' => $domain,
                ':username' => $username,
                ':id' => $id
            ]);

            servicesLog(
                $pdo,
                'hosting_service_updated',
                'Service #' . $id . ' updated. Domain: ' .
                $service['domain'] . ' -> ' . $domain
            );

            $pdo->commit();

            servicesRedirect('Service details updated successfully.');
        }

        if ($action === 'status') {
            $status = servicesPostString('status');

            if (!in_array($status, $allowedStatuses, true)) {
                throw new RuntimeException('Invalid service status.');
            }

            if ($service['status'] === 'terminated') {
                throw new RuntimeException('Terminated services cannot be reactivated.');
            }

            $pdo->beginTransaction();

            $update = $pdo->prepare(
                'UPDATE hosting_services
                 SET status = :status
                 WHERE id = :id'
            );

            $update->execute([
                ':status' => $status,
                ':id' => $id
            ]);

            servicesLog(
                $pdo,
                'hosting_service_status_changed',
                'Service #' . $id . ' status changed from ' .
                $service['status'] . ' to ' . $status
            );

            $pdo->commit();

            servicesRedirect('Service status updated successfully.');
        }

        throw new RuntimeException('Unknown action.');

    } catch (Throwable $e) {
        if ($pdo->inTransaction()) {
            $pdo->rollBack();
        }

        error_log('Hosting services error: ' . $e->getMessage());

        servicesRedirect(
            $e instanceof RuntimeException
                ? $e->getMessage()
                : 'An unexpected error occurred. Check the application log.',
            'error'
        );
    }
}

$flash = $_SESSION['services_flash'] ?? null;
unset($_SESSION['services_flash']);

$search = trim((string)($_GET['search'] ?? ''));
$statusFilter = (string)($_GET['status'] ?? '');

if (!in_array($statusFilter, array_merge([''], $allowedStatuses), true)) {
    $statusFilter = '';
}

$page = max(1, (int)($_GET['page'] ?? 1));
$perPage = 15;
$offset = ($page - 1) * $perPage;

$where = [];
$params = [];

if ($search !== '') {
    $where[] = '(hs.domain LIKE :search_domain
                 OR hs.username LIKE :search_username
                 OR u.name LIKE :search_name
                 OR u.email LIKE :search_email
                 OR CAST(hs.id AS CHAR) = :search_id)';

    $term = '%' . $search . '%';

    $params[':search_domain'] = $term;
    $params[':search_username'] = $term;
    $params[':search_name'] = $term;
    $params[':search_email'] = $term;
    $params[':search_id'] = $search;
}

if ($statusFilter !== '') {
    $where[] = 'hs.status = :status';
    $params[':status'] = $statusFilter;
}

$whereSql = $where ? 'WHERE ' . implode(' AND ', $where) : '';

$countStmt = $pdo->prepare(
    "SELECT COUNT(*)
     FROM hosting_services hs
     INNER JOIN users u ON u.id = hs.user_id
     INNER JOIN hosting_plans hp ON hp.id = hs.plan_id
     $whereSql"
);

$countStmt->execute($params);
$totalServices = (int)$countStmt->fetchColumn();
$totalPages = max(1, (int)ceil($totalServices / $perPage));

$page = min($page, $totalPages);
$offset = ($page - 1) * $perPage;

$listStmt = $pdo->prepare(
    "SELECT
        hs.id,
        hs.user_id,
        hs.plan_id,
        hs.order_id,
        hs.domain,
        hs.username,
        hs.status,
        hs.created_at,
        u.name AS client_name,
        u.email AS client_email,
        hp.name AS plan_name,
        o.status AS order_status
     FROM hosting_services hs
     INNER JOIN users u ON u.id = hs.user_id
     INNER JOIN hosting_plans hp ON hp.id = hs.plan_id
     LEFT JOIN orders o ON o.id = hs.order_id
     $whereSql
     ORDER BY hs.id DESC
     LIMIT $perPage OFFSET $offset"
);

$listStmt->execute($params);
$services = $listStmt->fetchAll();

$stats = $pdo->query(
    "SELECT
        COUNT(*) AS total,
        SUM(status = 'active') AS active,
        SUM(status = 'pending') AS pending,
        SUM(status = 'suspended') AS suspended,
        SUM(status = 'terminated') AS terminated_count
     FROM hosting_services"
)->fetch();

$baseQuery = [];

if ($search !== '') {
    $baseQuery['search'] = $search;
}

if ($statusFilter !== '') {
    $baseQuery['status'] = $statusFilter;
}

$queryString = http_build_query($baseQuery);

?>
<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Hosting Services | Hosting Lab</title>
<style>
:root {
    --bg:#0b1020;
    --panel:#121a2d;
    --panel2:#182238;
    --border:#26334d;
    --text:#edf2ff;
    --muted:#8e9bb5;
    --accent:#6685ff;
    --green:#36c995;
    --yellow:#e9b95d;
    --red:#ef737d;
}
*{box-sizing:border-box}
body{
    margin:0;background:var(--bg);color:var(--text);
    font-family:Inter,Segoe UI,Arial,sans-serif;
}
.layout{display:flex;min-height:100vh}
.sidebar{
    width:245px;background:#0e1628;border-right:1px solid var(--border);
    padding:28px 18px;flex-shrink:0;
}
.brand{font-size:21px;font-weight:800;margin-bottom:8px}
.brand span{color:var(--accent)}
.brand-sub{font-size:11px;color:var(--muted);margin-bottom:35px}
.nav-label{
    color:#63718d;font-size:10px;font-weight:700;
    letter-spacing:1.5px;margin:24px 10px 10px;
}
.nav-link{
    display:block;text-decoration:none;color:#aab6cc;
    padding:12px 14px;border-radius:9px;margin:4px 0;font-size:13px;
}
.nav-link:hover,.nav-link.active{
    background:#202d49;color:#fff;
}
.nav-link.active{box-shadow:inset 3px 0 var(--accent)}
.main{flex:1;min-width:0;padding:32px}
.topbar{
    display:flex;justify-content:space-between;align-items:center;
    gap:15px;margin-bottom:30px;
}
h1{font-size:27px;margin:0 0 7px}
.subtitle{font-size:13px;color:var(--muted)}
.admin{font-size:12px;color:var(--muted)}
.stats{
    display:grid;grid-template-columns:repeat(5,minmax(0,1fr));
    gap:14px;margin-bottom:24px;
}
.stat,.panel{
    background:var(--panel);border:1px solid var(--border);
    border-radius:13px;
}
.stat{padding:19px}
.stat-label{font-size:11px;color:var(--muted);margin-bottom:12px}
.stat-value{font-size:25px;font-weight:750}
.panel{padding:20px;margin-bottom:20px}
.panel-title{font-size:15px;font-weight:700;margin-bottom:17px}
.filters{
    display:flex;gap:10px;flex-wrap:wrap;align-items:center
}
input,select{
    background:#0d1526;color:var(--text);
    border:1px solid var(--border);border-radius:8px;
    padding:11px 12px;font-size:12px;outline:none;
}
input{min-width:220px;flex:1}
select{min-width:145px}
button,.button{
    border:0;border-radius:8px;padding:11px 15px;
    background:var(--accent);color:white;font-size:12px;
    font-weight:700;cursor:pointer;text-decoration:none;
}
button:hover,.button:hover{filter:brightness(1.12)}
.secondary{background:#263550}
.danger{background:#74343e}
.table-wrap{overflow-x:auto}
table{width:100%;border-collapse:collapse;min-width:950px}
th{
    color:#8391aa;text-align:left;font-size:10px;
    text-transform:uppercase;letter-spacing:.8px;
    padding:13px 10px;border-bottom:1px solid var(--border);
}
td{
    padding:15px 10px;border-bottom:1px solid #202c42;
    font-size:12px;vertical-align:middle;
}
tr:last-child td{border-bottom:0}
.domain{font-weight:700;color:#e8edff}
.small{font-size:11px;color:var(--muted);margin-top:5px}
.badge{
    display:inline-block;padding:6px 9px;border-radius:20px;
    font-size:10px;font-weight:700;text-transform:capitalize;
}
.active{background:#123b34;color:#57dfb2}
.pending{background:#45391e;color:#f0cb77}
.suspended{background:#462b35;color:#ff9ba6}
.terminated{background:#34394a;color:#b0bad0}
.actions{display:flex;gap:7px;flex-wrap:wrap}
.actions button{padding:8px 10px}
.flash{
    padding:13px 15px;border-radius:9px;margin-bottom:18px;
    font-size:12px;background:#15372f;color:#70e1b9;
}
.flash.error{background:#46242d;color:#ffabb2}
.pagination{
    display:flex;justify-content:space-between;align-items:center;
    margin-top:18px;color:var(--muted);font-size:12px;
}
.pagination a{color:#aebfff;text-decoration:none;margin-left:12px}
.empty{text-align:center;padding:45px;color:var(--muted)}
.footer{color:#64728c;font-size:11px;margin-top:25px}
@media(max-width:1050px){
    .stats{grid-template-columns:repeat(3,1fr)}
    .sidebar{width:205px}
}
@media(max-width:700px){
    .layout{display:block}
    .sidebar{width:100%;padding:17px}
    .brand-sub{margin-bottom:12px}
    .nav-label{display:none}
    .nav-link{display:inline-block;padding:9px}
    .main{padding:20px 13px}
    .stats{grid-template-columns:repeat(2,1fr)}
    .topbar{align-items:flex-start;flex-direction:column}
}
</style>
</head>
<body>
<div class="layout">
<aside class="sidebar">
    <div class="brand">HOSTING <span>LAB</span></div>
    <div class="brand-sub">ADMIN CONTROL PANEL</div>

    <div class="nav-label">MAIN MENU</div>
    <a class="nav-link" href="index.php">Dashboard</a>
    <a class="nav-link" href="plans.php">Hosting Plans</a>
    <a class="nav-link active" href="services.php">Services</a>
    <a class="nav-link" href="orders.php">Orders</a>
    <a class="nav-link" href="clients.php">Clients</a>

    <div class="nav-label">SYSTEM</div>
    <a class="nav-link" href="logs.php">Activity Logs</a>
    <a class="nav-link" href="settings.php">Settings</a>
    <a class="nav-link" href="logout.php">Logout</a>
</aside>

<main class="main">
    <header class="topbar">
        <div>
            <h1>Hosting Services</h1>
            <div class="subtitle">Manage client hosting services and service status.</div>
        </div>
        <div class="admin">
            Administrator: <?= servicesEscape($_SESSION['admin_name'] ?? 'Admin') ?>
        </div>
    </header>

    <?php if ($flash): ?>
        <div class="flash <?= ($flash['type'] ?? '') === 'error' ? 'error' : '' ?>">
            <?= servicesEscape($flash['message'] ?? '') ?>
        </div>
    <?php endif; ?>

    <section class="stats">
        <div class="stat">
            <div class="stat-label">TOTAL SERVICES</div>
            <div class="stat-value"><?= (int)$stats['total'] ?></div>
        </div>
        <div class="stat">
            <div class="stat-label">ACTIVE</div>
            <div class="stat-value"><?= (int)$stats['active'] ?></div>
        </div>
        <div class="stat">
            <div class="stat-label">PENDING</div>
            <div class="stat-value"><?= (int)$stats['pending'] ?></div>
        </div>
        <div class="stat">
            <div class="stat-label">SUSPENDED</div>
            <div class="stat-value"><?= (int)$stats['suspended'] ?></div>
        </div>
        <div class="stat">
            <div class="stat-label">TERMINATED</div>
            <div class="stat-value"><?= (int)$stats['terminated_count'] ?></div>
        </div>
    </section>

    <section class="panel">
        <div class="panel-title">Search and filters</div>
        <form class="filters" method="get">
            <input
                type="search"
                name="search"
                placeholder="Search domain, username, client or ID..."
                value="<?= servicesEscape($search) ?>"
            >
            <select name="status">
                <option value="">All statuses</option>
                <?php foreach ($allowedStatuses as $status): ?>
                    <option value="<?= servicesEscape($status) ?>"
                        <?= $statusFilter === $status ? 'selected' : '' ?>>
                        <?= ucfirst($status) ?>
                    </option>
                <?php endforeach; ?>
            </select>
            <button type="submit">Apply filters</button>
            <a class="button secondary" href="services.php">Reset</a>
        </form>
    </section>

    <section class="panel">
        <div class="panel-title">Services directory</div>

        <div class="table-wrap">
        <table>
            <thead>
                <tr>
                    <th>Service</th>
                    <th>Client</th>
                    <th>Hosting Plan</th>
                    <th>Order</th>
                    <th>Status</th>
                    <th>Created</th>
                    <th>Actions</th>
                </tr>
            </thead>
            <tbody>
            <?php if (!$services): ?>
                <tr>
                    <td colspan="7" class="empty">No hosting services found.</td>
                </tr>
            <?php else: ?>
                <?php foreach ($services as $service): ?>
                <tr>
                    <td>
                        <div class="domain"><?= servicesEscape($service['domain']) ?></div>
                        <div class="small">
                            #<?= (int)$service['id'] ?> Â·
                            <?= servicesEscape($service['username']) ?>
                        </div>
                    </td>
                    <td>
                        <strong><?= servicesEscape($service['client_name']) ?></strong>
                        <div class="small"><?= servicesEscape($service['client_email']) ?></div>
                    </td>
                    <td><?= servicesEscape($service['plan_name']) ?></td>
                    <td>
                        <?= $service['order_id'] !== null
                            ? '#' . (int)$service['order_id']
                            : 'â€”' ?>
                        <?php if ($service['order_status']): ?>
                            <div class="small"><?= servicesEscape($service['order_status']) ?></div>
                        <?php endif; ?>
                    </td>
                    <td>
                        <span class="badge <?= servicesEscape($service['status']) ?>">
                            <?= servicesEscape($service['status']) ?>
                        </span>
                    </td>
                    <td><?= servicesEscape($service['created_at']) ?></td>
                    <td>
                        <div class="actions">
                            <button type="button" class="secondary"
                                onclick="editService(
                                    <?= (int)$service['id'] ?>,
                                    <?= htmlspecialchars(json_encode($service['domain']), ENT_QUOTES, 'UTF-8') ?>,
                                    <?= htmlspecialchars(json_encode($service['username']), ENT_QUOTES, 'UTF-8') ?>
                                )">Edit</button>

                            <?php if ($service['status'] !== 'terminated'): ?>
                            <form method="post" onsubmit="return confirm('Change status of this service?')">
                                <input type="hidden" name="csrf_token"
                                    value="<?= servicesEscape($_SESSION['admin_csrf']) ?>">
                                <input type="hidden" name="action" value="status">
                                <input type="hidden" name="id" value="<?= (int)$service['id'] ?>">
                                <input type="hidden" name="status"
                                    value="<?= $service['status'] === 'active' ? 'suspended' : 'active' ?>">
                                <button type="submit" class="<?= $service['status'] === 'active' ? 'danger' : '' ?>">
                                    <?= $service['status'] === 'active' ? 'Suspend' : 'Activate' ?>
                                </button>
                            </form>

                            <form method="post" onsubmit="return confirm('Terminate this service permanently? This cannot be undone here.')">
                                <input type="hidden" name="csrf_token"
                                    value="<?= servicesEscape($_SESSION['admin_csrf']) ?>">
                                <input type="hidden" name="action" value="status">
                                <input type="hidden" name="id" value="<?= (int)$service['id'] ?>">
                                <input type="hidden" name="status" value="terminated">
                                <button type="submit" class="danger">Terminate</button>
                            </form>
                            <?php endif; ?>
                        </div>
                    </td>
                </tr>
                <?php endforeach; ?>
            <?php endif; ?>
            </tbody>
        </table>
        </div>

        <div class="pagination">
            <span>
                Showing <?= count($services) ?> of <?= $totalServices ?> services
            </span>
            <span>
                <?php if ($page > 1): ?>
                    <a href="?<?= servicesEscape($queryString) ?>&page=<?= $page - 1 ?>">Previous</a>
                <?php endif; ?>
                Page <?= $page ?> / <?= $totalPages ?>
                <?php if ($page < $totalPages): ?>
                    <a href="?<?= servicesEscape($queryString) ?>&page=<?= $page + 1 ?>">Next</a>
                <?php endif; ?>
            </span>
        </div>
    </section>

    <div class="footer">Hosting Lab Â· Services Management</div>
</main>
</div>

<div id="editModal" style="display:none;position:fixed;inset:0;background:#000b;z-index:10;align-items:center;justify-content:center;padding:20px">
    <div class="panel" style="width:100%;max-width:460px">
        <div class="panel-title">Edit hosting service</div>
        <form method="post">
            <input type="hidden" name="csrf_token"
                value="<?= servicesEscape($_SESSION['admin_csrf']) ?>">
            <input type="hidden" name="action" value="update">
            <input type="hidden" name="id" id="editId">

            <label class="small">Domain</label>
            <input style="width:100%;margin:8px 0 16px" name="domain" id="editDomain" maxlength="190" required>

            <label class="small">Username</label>
            <input style="width:100%;margin:8px 0 18px" name="username" id="editUsername" maxlength="100" required>

            <div class="actions">
                <button type="submit">Save changes</button>
                <button type="button" class="secondary" onclick="closeEdit()">Cancel</button>
            </div>
        </form>
    </div>
</div>

<script>
function editService(id, domain, username) {
    document.getElementById('editId').value = id;
    document.getElementById('editDomain').value = domain;
    document.getElementById('editUsername').value = username;
    document.getElementById('editModal').style.display = 'flex';
}

function closeEdit() {
    document.getElementById('editModal').style.display = 'none';
}

document.getElementById('editModal').addEventListener('click', function(event) {
    if (event.target === this) closeEdit();
});
</script>
</body>
</html>

===== client\orders.php =====
<?php
declare(strict_types=1);
require_once __DIR__ . '/../includes/public.php';

$client = hl_require_client($pdo);
$pageTitle = 'My Orders';

$stmt = $pdo->prepare(
    'SELECT o.id, o.amount, o.status, o.created_at, hp.name AS plan_name, hp.billing_cycle
     FROM orders o INNER JOIN hosting_plans hp ON hp.id = o.plan_id
     WHERE o.user_id = :id ORDER BY o.id DESC'
);
$stmt->execute([':id' => $client['id']]);
$orders = $stmt->fetchAll();

require __DIR__ . '/../includes/site_header.php';
?>
<section class="dashboard-hero"><div class="container dashboard-heading"><div><div class="eyebrow">CLIENT AREA</div><h1>My <span>orders.</span></h1><p>Review your order history and current order status.</p></div><a class="button" href="<?= hl_url('plans.php') ?>">New Order <span>â†’</span></a></div></section>
<section class="section dashboard-section"><div class="container">
    <div class="dashboard-nav"><a href="<?= hl_url('client/index.php') ?>">Overview</a><a class="dash-active" href="<?= hl_url('client/orders.php') ?>">My Orders</a><a href="<?= hl_url('client/services.php') ?>">My Services</a><a href="<?= hl_url('cpanel/index.php') ?>">Hosting Control Panel</a><a href="<?= hl_url('logout.php') ?>">Logout</a></div>
    <div class="content-card"><div class="card-heading"><div><div class="eyebrow">ORDER HISTORY</div><h2>All orders</h2></div></div>
    <?php if (isset($_GET['created'])): ?><div class="alert alert-success">Your order was created and is pending review/payment.</div><?php endif; ?>
    <?php if ($orders): ?>
    <div class="table-wrap"><table class="data-table"><thead><tr><th>Order</th><th>Plan</th><th>Billing</th><th>Amount</th><th>Status</th><th>Created</th></tr></thead><tbody>
    <?php foreach ($orders as $order): ?><tr><td>#<?= (int)$order['id'] ?></td><td><?= hl_escape($order['plan_name']) ?></td><td><?= hl_escape(ucfirst($order['billing_cycle'])) ?></td><td><?= hl_money($order['amount']) ?></td><td><span class="status status-<?= hl_escape($order['status']) ?>"><?= hl_escape(ucfirst($order['status'])) ?></span></td><td><?= hl_escape($order['created_at']) ?></td></tr><?php endforeach; ?>
    </tbody></table></div>
    <?php else: ?><div class="empty-state">No orders yet. <a href="<?= hl_url('plans.php') ?>">Choose your first plan.</a></div><?php endif; ?>
    </div>
</div></section>
<?php require __DIR__ . '/../includes/site_footer.php'; ?>

===== client\services.php =====
<?php
declare(strict_types=1);
require_once __DIR__ . '/../includes/public.php';

$client = hl_require_client($pdo);
$pageTitle = 'My Services';

$stmt = $pdo->prepare(
    'SELECT hs.id, hs.domain, hs.username, hs.status, hs.created_at, hp.name AS plan_name
     FROM hosting_services hs INNER JOIN hosting_plans hp ON hp.id = hs.plan_id
     WHERE hs.user_id = :id ORDER BY hs.id DESC'
);
$stmt->execute([':id' => $client['id']]);
$services = $stmt->fetchAll();

require __DIR__ . '/../includes/site_header.php';
?>
<section class="dashboard-hero"><div class="container dashboard-heading"><div><div class="eyebrow">CLIENT AREA</div><h1>My <span>services.</span></h1><p>View the hosting services assigned to your account.</p></div><a class="button" href="<?= hl_url('plans.php') ?>">Browse Plans <span>â†’</span></a></div></section>
<section class="section dashboard-section"><div class="container">
    <div class="dashboard-nav"><a href="<?= hl_url('client/index.php') ?>">Overview</a><a href="<?= hl_url('client/orders.php') ?>">My Orders</a><a class="dash-active" href="<?= hl_url('client/services.php') ?>">My Services</a><a href="<?= hl_url('cpanel/index.php') ?>">Hosting Control Panel</a><a href="<?= hl_url('logout.php') ?>">Logout</a></div>
    <div class="content-card"><div class="card-heading"><div><div class="eyebrow">HOSTING SERVICES</div><h2>Assigned services</h2></div></div>
    <?php if ($services): ?>
    <div class="table-wrap"><table class="data-table"><thead><tr><th>Service</th><th>Plan</th><th>Domain</th><th>Username</th><th>Status</th><th>Created</th></tr></thead><tbody>
    <?php foreach ($services as $service): ?><tr><td>#<?= (int)$service['id'] ?></td><td><?= hl_escape($service['plan_name']) ?></td><td><?= hl_escape($service['domain']) ?></td><td><?= hl_escape($service['username']) ?></td><td><span class="status status-<?= hl_escape($service['status']) ?>"><?= hl_escape(ucfirst($service['status'])) ?></span></td><td><?= hl_escape($service['created_at']) ?></td></tr><?php endforeach; ?>
    </tbody></table></div>
    <?php else: ?><div class="empty-state">No hosting services have been assigned to your account yet. New services will appear here when provisioned.</div><?php endif; ?>
    </div>
</div></section>
<?php require __DIR__ . '/../includes/site_footer.php'; ?>

===== plans.php =====
<?php
declare(strict_types=1);
require_once __DIR__ . '/includes/public.php';

$pageTitle = 'Hosting Plans';
$activeNav = 'plans';

$stmt = $pdo->query("SELECT * FROM hosting_plans WHERE status = 'active' ORDER BY price ASC, id ASC");
$plans = $stmt->fetchAll();

require __DIR__ . '/includes/site_header.php';
?>
<section class="page-hero">
    <div class="container">
        <div class="eyebrow">FIND YOUR PERFECT FIT</div>
        <h1>Hosting plans.<br><span>Clear and simple.</span></h1>
        <p>Compare the available plans and choose the resources that match your project.</p>
    </div>
</section>
<section class="section">
    <div class="container">
        <?php if ($plans): ?>
            <div class="pricing-grid pricing-grid-all">
                <?php foreach ($plans as $i => $plan): ?>
                    <article class="price-card <?= $i === 1 ? 'price-featured' : '' ?>">
                        <div class="plan-kicker">HOSTING PLAN</div>
                        <h3><?= hl_escape($plan['name']) ?></h3>
                        <p class="plan-description"><?= hl_escape($plan['description'] ?: 'A reliable foundation for your next project.') ?></p>
                        <div class="price"><?= hl_money($plan['price']) ?><small>/ <?= hl_escape(hl_cycle($plan['billing_cycle'])) ?></small></div>
                        <ul class="plan-features">
                            <?php foreach (hl_plan_specs($plan) as $spec): ?>
                                <li><span>âœ“</span><?= hl_escape($spec) ?></li>
                            <?php endforeach; ?>
                        </ul>
                        <a class="button <?= $i === 1 ? '' : 'button-outline' ?> button-full" href="<?= hl_url('order.php?plan=' . (int)$plan['id']) ?>">Choose Plan <span>â†’</span></a>
                    </article>
                <?php endforeach; ?>
            </div>
        <?php else: ?>
            <div class="empty-state">There are no active plans at the moment.</div>
        <?php endif; ?>
    </div>
</section>
<?php require __DIR__ . '/includes/site_footer.php'; ?>

===== DOMAIN / USERNAME REFERENCES =====

Path                                                             LineNumber Line
----                                                             ---------- ----
D:\Program Files\XAMPP\htdocs\hosting\admin\clients.php                 294         (SELECT COUNT(*) FROM hosting_se...
D:\Program Files\XAMPP\htdocs\hosting\admin\index.php                    45         "SELECT COUNT(*) FROM hosting_se...
D:\Program Files\XAMPP\htdocs\hosting\admin\index.php                    49         "SELECT COUNT(*) FROM hosting_se...
D:\Program Files\XAMPP\htdocs\hosting\admin\index.php                    53         "SELECT COUNT(*) FROM hosting_se...
D:\Program Files\XAMPP\htdocs\hosting\admin\login.php                   202                 autocomplete="username"
D:\Program Files\XAMPP\htdocs\hosting\admin\logout.php                   51             'domain' => $params['domain'],
D:\Program Files\XAMPP\htdocs\hosting\admin\orders.php                  264             hp.domains_limit,
D:\Program Files\XAMPP\htdocs\hosting\admin\orders.php                 1070                         <div class="deta...
D:\Program Files\XAMPP\htdocs\hosting\admin\orders.php                 1071                         <div class="deta...
D:\Program Files\XAMPP\htdocs\hosting\admin\plans.php                   104         'domains_limit' => readInteger('...
D:\Program Files\XAMPP\htdocs\hosting\admin\plans.php                   158                      domains_limit, data...
D:\Program Files\XAMPP\htdocs\hosting\admin\plans.php                   162                      :domains, :database...
D:\Program Files\XAMPP\htdocs\hosting\admin\plans.php                   171                     ':domains' => $data[...
D:\Program Files\XAMPP\htdocs\hosting\admin\plans.php                   201                     domains_limit = :dom...
D:\Program Files\XAMPP\htdocs\hosting\admin\plans.php                   215                 ':domains' => $data['dom...
D:\Program Files\XAMPP\htdocs\hosting\admin\plans.php                   295                 'SELECT COUNT(*) FROM ho...
D:\Program Files\XAMPP\htdocs\hosting\admin\plans.php                   878                     <label for="domains_...
D:\Program Files\XAMPP\htdocs\hosting\admin\plans.php                   879                     <input id="domains_l...
D:\Program Files\XAMPP\htdocs\hosting\admin\plans.php                   881                            value="<?= h(...
D:\Program Files\XAMPP\htdocs\hosting\admin\plans.php                   958                 <th>Domains</th>
D:\Program Files\XAMPP\htdocs\hosting\admin\plans.php                   987                 <td><?= h(formatRegularL...
D:\Program Files\XAMPP\htdocs\hosting\admin\services.php                 66             'SELECT id, domain, username...
D:\Program Files\XAMPP\htdocs\hosting\admin\services.php                 67              FROM hosting_services
D:\Program Files\XAMPP\htdocs\hosting\admin\services.php                 79             $domain = servicesPostString...
D:\Program Files\XAMPP\htdocs\hosting\admin\services.php                 80             $username = servicesPostStri...
D:\Program Files\XAMPP\htdocs\hosting\admin\services.php                 83                 $domain === '' ||
D:\Program Files\XAMPP\htdocs\hosting\admin\services.php                 84                 mb_strlen($domain) > 190 ||
D:\Program Files\XAMPP\htdocs\hosting\admin\services.php                 85                 preg_match('/[\s\/\\\\]/...
D:\Program Files\XAMPP\htdocs\hosting\admin\services.php                 87                 throw new RuntimeExcepti...
D:\Program Files\XAMPP\htdocs\hosting\admin\services.php                 91                 $username === '' ||
D:\Program Files\XAMPP\htdocs\hosting\admin\services.php                 92                 mb_strlen($username) > 100
D:\Program Files\XAMPP\htdocs\hosting\admin\services.php                 94                 throw new RuntimeExcepti...
D:\Program Files\XAMPP\htdocs\hosting\admin\services.php                100                 'UPDATE hosting_services
D:\Program Files\XAMPP\htdocs\hosting\admin\services.php                101                  SET domain = :domain, u...
D:\Program Files\XAMPP\htdocs\hosting\admin\services.php                106                 ':domain' => $domain,
D:\Program Files\XAMPP\htdocs\hosting\admin\services.php                107                 ':username' => $username,
D:\Program Files\XAMPP\htdocs\hosting\admin\services.php                114                 'Service #' . $id . ' up...
D:\Program Files\XAMPP\htdocs\hosting\admin\services.php                115                 $service['domain'] . ' -...
D:\Program Files\XAMPP\htdocs\hosting\admin\services.php                137                 'UPDATE hosting_services
D:\Program Files\XAMPP\htdocs\hosting\admin\services.php                195     $where[] = '(hs.domain LIKE :search_...
D:\Program Files\XAMPP\htdocs\hosting\admin\services.php                196                  OR hs.username LIKE :se...
D:\Program Files\XAMPP\htdocs\hosting\admin\services.php                203     $params[':search_domain'] = $term;
D:\Program Files\XAMPP\htdocs\hosting\admin\services.php                204     $params[':search_username'] = $term;
D:\Program Files\XAMPP\htdocs\hosting\admin\services.php                219      FROM hosting_services hs
D:\Program Files\XAMPP\htdocs\hosting\admin\services.php                238         hs.domain,
D:\Program Files\XAMPP\htdocs\hosting\admin\services.php                239         hs.username,
D:\Program Files\XAMPP\htdocs\hosting\admin\services.php                246      FROM hosting_services hs
D:\Program Files\XAMPP\htdocs\hosting\admin\services.php                265      FROM hosting_services"
D:\Program Files\XAMPP\htdocs\hosting\admin\services.php                376 .domain{font-weight:700;color:#e8edff}
D:\Program Files\XAMPP\htdocs\hosting\admin\services.php                481                 placeholder="Search doma...
D:\Program Files\XAMPP\htdocs\hosting\admin\services.php                523                         <div class="doma...
D:\Program Files\XAMPP\htdocs\hosting\admin\services.php                526                             <?= services...
D:\Program Files\XAMPP\htdocs\hosting\admin\services.php                553                                     <?= ...
D:\Program Files\XAMPP\htdocs\hosting\admin\services.php                554                                     <?= ...
D:\Program Files\XAMPP\htdocs\hosting\admin\services.php                617             <label class="small">Domain<...
D:\Program Files\XAMPP\htdocs\hosting\admin\services.php                618             <input style="width:100%;mar...
D:\Program Files\XAMPP\htdocs\hosting\admin\services.php                620             <label class="small">Usernam...
D:\Program Files\XAMPP\htdocs\hosting\admin\services.php                621             <input style="width:100%;mar...
D:\Program Files\XAMPP\htdocs\hosting\admin\services.php                632 function editService(id, domain, usernam...
D:\Program Files\XAMPP\htdocs\hosting\admin\services.php                634     document.getElementById('editDomain'...
D:\Program Files\XAMPP\htdocs\hosting\admin\services.php                635     document.getElementById('editUsernam...
D:\Program Files\XAMPP\htdocs\hosting\client\index.php                   12 $countServices = $pdo->prepare('SELECT C...
D:\Program Files\XAMPP\htdocs\hosting\client\services.php                 9     'SELECT hs.id, hs.domain, hs.usernam...
D:\Program Files\XAMPP\htdocs\hosting\client\services.php                10      FROM hosting_services hs INNER JOIN...
D:\Program Files\XAMPP\htdocs\hosting\client\services.php                23     <div class="table-wrap"><table class...
D:\Program Files\XAMPP\htdocs\hosting\client\services.php                24     <?php foreach ($services as $service...
D:\Program Files\XAMPP\htdocs\hosting\client\services.php                26     <?php else: ?><div class="empty-stat...
D:\Program Files\XAMPP\htdocs\hosting\config\database.php                 6 $username = 'root';
D:\Program Files\XAMPP\htdocs\hosting\config\database.php                12         $username,
D:\Program Files\XAMPP\htdocs\hosting\cpanel\domains.php                  4 $moduleKey = 'domains';
D:\Program Files\XAMPP\htdocs\hosting\cpanel\domains.php                  5 $moduleTitle = 'Domains';
D:\Program Files\XAMPP\htdocs\hosting\cpanel\domains.php                  6 $moduleDescription = 'Manage domains ass...
D:\Program Files\XAMPP\htdocs\hosting\cpanel\index.php                   10     'SELECT COUNT(*) FROM hosting_servic...
D:\Program Files\XAMPP\htdocs\hosting\cpanel\index.php                   16     "SELECT COUNT(*) FROM hosting_services
D:\Program Files\XAMPP\htdocs\hosting\cpanel\index.php                   29     'SELECT hs.id, hs.domain, hs.usernam...
D:\Program Files\XAMPP\htdocs\hosting\cpanel\index.php                   31      FROM hosting_services hs
D:\Program Files\XAMPP\htdocs\hosting\cpanel\index.php                   91                         <th>Domain</th>
D:\Program Files\XAMPP\htdocs\hosting\cpanel\index.php                  100                         <td><?= hl_escap...
D:\Program Files\XAMPP\htdocs\hosting\cpanel\services.php                10     'SELECT hs.id, hs.domain, hs.usernam...
D:\Program Files\XAMPP\htdocs\hosting\cpanel\services.php                13             hp.domains_limit, hp.databas...
D:\Program Files\XAMPP\htdocs\hosting\cpanel\services.php                15      FROM hosting_services hs
D:\Program Files\XAMPP\htdocs\hosting\cpanel\services.php                52                 <h3><?= hl_escape($servi...
D:\Program Files\XAMPP\htdocs\hosting\cpanel\services.php                56                     <div><span>Username<...
D:\Program Files\XAMPP\htdocs\hosting\cpanel\services.php                59                     <div><span>Domains</...
D:\Program Files\XAMPP\htdocs\hosting\cpanel\ssl.php                      6 $moduleDescription = 'View and manage SS...
D:\Program Files\XAMPP\htdocs\hosting\cpanel\includes\header.php         35             <a class="<?= $activePage ==...
D:\Program Files\XAMPP\htdocs\hosting\cpanel\includes\header.php         36                 <span>◎</span> Domains
D:\Program Files\XAMPP\htdocs\hosting\includes\public.php               129         (int)$plan['domains_limit'] . ' ...
