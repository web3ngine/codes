<?php
declare(strict_types=1);

require_once __DIR__ . '/../includes/auth.php';
require_once __DIR__ . '/../config/database.php';

requireAdmin($pdo);

function ordersRedirect(string $message, string $type = 'success'): never
{
    $_SESSION['orders_flash'] = [
        'message' => $message,
        'type' => $type,
    ];

    header('Location: orders.php');
    exit;
}

function ordersLog(PDO $pdo, string $action, string $details): void
{
    $stmt = $pdo->prepare(
        'INSERT INTO activity_logs (user_id, action, details, ip_address)
         VALUES (:user_id, :action, :details, :ip)'
    );

    $stmt->execute([
        ':user_id' => (int)$_SESSION['admin_id'],
        ':action' => $action,
        ':details' => $details,
        ':ip' => $_SERVER['REMOTE_ADDR'] ?? null,
    ]);
}

$allowedStatuses = ['pending', 'paid', 'cancelled', 'failed'];

if ($_SERVER['REQUEST_METHOD'] === 'POST') {
    $csrf = (string)($_POST['csrf_token'] ?? '');

    if (
        empty($_SESSION['admin_csrf']) ||
        !hash_equals($_SESSION['admin_csrf'], $csrf)
    ) {
        ordersRedirect('Security token expired. Please try again.', 'error');
    }

    $action = (string)($_POST['action'] ?? '');

    try {
        if ($action !== 'update_status') {
            throw new RuntimeException('Unknown action.');
        }

        $orderId = filter_var(
            $_POST['order_id'] ?? null,
            FILTER_VALIDATE_INT
        );

        if (!$orderId || $orderId < 1) {
            throw new RuntimeException('Invalid order ID.');
        }

        $newStatus = (string)($_POST['status'] ?? '');

        if (!in_array($newStatus, $allowedStatuses, true)) {
            throw new RuntimeException('Invalid order status.');
        }

        $pdo->beginTransaction();

        $find = $pdo->prepare(
            'SELECT id, user_id, plan_id, amount, status
             FROM orders
             WHERE id = :id
             FOR UPDATE'
        );

        $find->execute([':id' => $orderId]);
        $order = $find->fetch();

        if (!$order) {
            $pdo->rollBack();
            throw new RuntimeException('Order was not found.');
        }

        $oldStatus = (string)$order['status'];

        if ($oldStatus === $newStatus) {
            $pdo->rollBack();
            ordersRedirect('The order already has this status.', 'error');
        }

        $update = $pdo->prepare(
            'UPDATE orders
             SET status = :status
             WHERE id = :id'
        );

        $update->execute([
            ':status' => $newStatus,
            ':id' => $orderId,
        ]);

        ordersLog(
            $pdo,
            'order_status_changed',
            'Order #' . $orderId .
            ' status changed from ' . $oldStatus .
            ' to ' . $newStatus .
            '. Amount: ' . number_format((float)$order['amount'], 2, '.', '') .
            '. Client ID: ' . (int)$order['user_id'] .
            '. Plan ID: ' . (int)$order['plan_id'] . '.'
        );

        $pdo->commit();

        ordersRedirect('Order #' . $orderId . ' status updated successfully.');

    } catch (PDOException $e) {
        if ($pdo->inTransaction()) {
            $pdo->rollBack();
        }

        error_log($e->getMessage());
        ordersRedirect('A database error occurred. Check the server error log.', 'error');

    } catch (RuntimeException $e) {
        if ($pdo->inTransaction()) {
            $pdo->rollBack();
        }

        ordersRedirect($e->getMessage(), 'error');
    }
}

/* ---------------------------------------------------------
   Filters and pagination
--------------------------------------------------------- */

$flash = $_SESSION['orders_flash'] ?? null;
unset($_SESSION['orders_flash']);

$search = trim((string)($_GET['q'] ?? ''));
$statusFilter = (string)($_GET['status'] ?? '');

if (!in_array($statusFilter, array_merge(['all'], $allowedStatuses), true)) {
    $statusFilter = 'all';
}

$page = max(1, (int)($_GET['page'] ?? 1));
$perPage = 15;

$where = 'WHERE 1 = 1';
$params = [];

if ($search !== '') {
    $where .= "
        AND (
            CAST(o.id AS CHAR) LIKE :order_id
            OR u.name LIKE :client_name
            OR u.email LIKE :client_email
            OR hp.name LIKE :plan_name
        )
    ";

    $term = '%' . $search . '%';

    $params[':order_id'] = $term;
    $params[':client_name'] = $term;
    $params[':client_email'] = $term;
    $params[':plan_name'] = $term;
}

if ($statusFilter !== 'all') {
    $where .= ' AND o.status = :status';
    $params[':status'] = $statusFilter;
}

$countStmt = $pdo->prepare(
    "SELECT COUNT(*)
     FROM orders o
     INNER JOIN users u ON u.id = o.user_id
     INNER JOIN hosting_plans hp ON hp.id = o.plan_id
     $where"
);

$countStmt->execute($params);
$totalOrders = (int)$countStmt->fetchColumn();

$totalPages = max(1, (int)ceil($totalOrders / $perPage));
$page = min($page, $totalPages);
$offset = ($page - 1) * $perPage;

$listStmt = $pdo->prepare(
    "SELECT
        o.id,
        o.user_id,
        o.plan_id,
        o.amount,
        o.status,
        o.created_at,
        u.name AS client_name,
        u.email AS client_email,
        hp.name AS plan_name,
        hp.billing_cycle
     FROM orders o
     INNER JOIN users u ON u.id = o.user_id
     INNER JOIN hosting_plans hp ON hp.id = o.plan_id
     $where
     ORDER BY o.id DESC
     LIMIT :limit OFFSET :offset"
);

foreach ($params as $key => $value) {
    $listStmt->bindValue($key, $value, PDO::PARAM_STR);
}

$listStmt->bindValue(':limit', $perPage, PDO::PARAM_INT);
$listStmt->bindValue(':offset', $offset, PDO::PARAM_INT);
$listStmt->execute();

$orders = $listStmt->fetchAll();

/* ---------------------------------------------------------
   Dashboard statistics
--------------------------------------------------------- */

$stats = $pdo->query(
    "SELECT
        COUNT(*) AS total,
        SUM(status = 'pending') AS pending_count,
        SUM(status = 'paid') AS paid_count,
        SUM(status = 'cancelled') AS cancelled_count,
        SUM(status = 'failed') AS failed_count,
        COALESCE(SUM(CASE WHEN status = 'paid' THEN amount ELSE 0 END), 0) AS revenue
     FROM orders"
)->fetch();

/* ---------------------------------------------------------
   Selected order details
--------------------------------------------------------- */

$selectedOrder = null;
$viewId = filter_var($_GET['view'] ?? null, FILTER_VALIDATE_INT);

if ($viewId && $viewId > 0) {
    $detailStmt = $pdo->prepare(
        "SELECT
            o.id,
            o.user_id,
            o.plan_id,
            o.amount,
            o.status,
            o.created_at,
            u.name AS client_name,
            u.email AS client_email,
            u.status AS client_status,
            hp.name AS plan_name,
            hp.description AS plan_description,
            hp.price AS plan_price,
            hp.billing_cycle,
            hp.disk_limit_mb,
            hp.bandwidth_limit_mb,
            hp.domains_limit,
            hp.databases_limit,
            hp.ftp_accounts_limit
         FROM orders o
         INNER JOIN users u ON u.id = o.user_id
         INNER JOIN hosting_plans hp ON hp.id = o.plan_id
         WHERE o.id = :id
         LIMIT 1"
    );

    $detailStmt->execute([':id' => $viewId]);
    $selectedOrder = $detailStmt->fetch() ?: null;
}

function ordersPageUrl(int $page, string $search, string $status): string
{
    $query = ['page' => $page];

    if ($search !== '') {
        $query['q'] = $search;
    }

    if ($status !== 'all') {
        $query['status'] = $status;
    }

    return 'orders.php?' . http_build_query($query);
}

function orderStatusLabel(string $status): string
{
    return match ($status) {
        'pending' => 'Pending',
        'paid' => 'Paid',
        'cancelled' => 'Cancelled',
        'failed' => 'Failed',
        default => ucfirst($status),
    };
}

?>
<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Orders | Hosting Lab</title>

<style>
:root {
    color-scheme: dark;
    --bg: #090d16;
    --panel: #111827;
    --panel2: #172235;
    --border: #263449;
    --text: #edf2fa;
    --muted: #91a0b5;
    --accent: #6c8cff;
    --green: #35d49a;
    --red: #ff7185;
    --yellow: #f5c76b;
}

* { box-sizing: border-box; }

body {
    margin: 0;
    min-height: 100vh;
    background:
        radial-gradient(circle at 85% 0%, #17264a 0, transparent 32%),
        var(--bg);
    color: var(--text);
    font-family: Inter, "Segoe UI", Arial, sans-serif;
}

.layout { display: flex; min-height: 100vh; }

.sidebar {
    width: 245px;
    padding: 28px 18px;
    background: rgba(12,18,30,.96);
    border-right: 1px solid var(--border);
    flex-shrink: 0;
}

.brand {
    font-size: 21px;
    font-weight: 800;
    margin: 0 10px 8px;
}

.brand span { color: var(--accent); }

.brand-subtitle {
    color: var(--muted);
    font-size: 11px;
    margin: 0 10px 38px;
    letter-spacing: 1.5px;
    text-transform: uppercase;
}

.nav-label {
    color: #63738a;
    font-size: 10px;
    font-weight: 700;
    letter-spacing: 1.5px;
    margin: 25px 10px 12px;
    text-transform: uppercase;
}

.nav-link {
    display: block;
    padding: 12px 13px;
    margin: 5px 0;
    color: var(--muted);
    text-decoration: none;
    border-radius: 9px;
    font-size: 13px;
}

.nav-link:hover, .nav-link.active {
    color: white;
    background: #202e49;
}

.nav-link.active { box-shadow: inset 3px 0 var(--accent); }

.main {
    flex: 1;
    min-width: 0;
    padding: 35px;
}

.topbar {
    display: flex;
    justify-content: space-between;
    align-items: center;
    gap: 20px;
    margin-bottom: 30px;
}

.breadcrumb {
    color: var(--muted);
    font-size: 12px;
    margin-bottom: 9px;
}

h1 {
    margin: 0;
    font-size: 29px;
    letter-spacing: -1px;
}

.subtitle {
    color: var(--muted);
    margin-top: 9px;
    font-size: 13px;
}

.environment {
    padding: 9px 13px;
    border: 1px solid #235b4a;
    background: #102c26;
    color: var(--green);
    border-radius: 30px;
    font-size: 11px;
    white-space: nowrap;
}

.environment::before {
    content: "";
    display: inline-block;
    width: 7px;
    height: 7px;
    margin-right: 8px;
    background: var(--green);
    border-radius: 50%;
}

.stats {
    display: grid;
    grid-template-columns: repeat(4, minmax(0, 1fr));
    gap: 15px;
    margin-bottom: 25px;
}

.stat-card {
    padding: 20px;
    border: 1px solid var(--border);
    border-radius: 13px;
    background: linear-gradient(145deg, #152137, #101725);
}

.stat-label { color: var(--muted); font-size: 12px; }

.stat-value {
    margin-top: 12px;
    font-size: 26px;
    font-weight: 750;
}

.stat-value.green { color: var(--green); }
.stat-value.yellow { color: var(--yellow); }
.stat-value.red { color: var(--red); }

.panel {
    border: 1px solid var(--border);
    border-radius: 14px;
    overflow: hidden;
    background: rgba(17,24,39,.94);
    margin-bottom: 24px;
}

.panel-header {
    padding: 20px 23px;
    border-bottom: 1px solid var(--border);
}

.panel-header h2 {
    margin: 0;
    font-size: 16px;
}

.panel-header p {
    margin: 6px 0 0;
    color: var(--muted);
    font-size: 12px;
}

.flash {
    padding: 14px 17px;
    margin-bottom: 20px;
    border: 1px solid var(--border);
    border-radius: 9px;
    font-size: 13px;
}

.flash.success {
    background: #102c26;
    border-color: #235b4a;
    color: var(--green);
}

.flash.error {
    background: #351b25;
    border-color: #69303f;
    color: #ff9aaa;
}

.notice {
    padding: 13px 16px;
    margin: 18px 22px;
    border: 1px solid #66502a;
    border-radius: 9px;
    background: #2b2417;
    color: #f5d18a;
    font-size: 12px;
    line-height: 1.6;
}

.toolbar {
    display: flex;
    justify-content: space-between;
    align-items: center;
    gap: 14px;
    padding: 18px 22px;
    flex-wrap: wrap;
}

.filters {
    display: flex;
    gap: 8px;
    flex: 1;
    flex-wrap: wrap;
}

.filters input,
.filters select {
    padding: 10px 12px;
    border: 1px solid var(--border);
    border-radius: 8px;
    background: #0b1321;
    color: var(--text);
    outline: none;
    font: inherit;
    font-size: 12px;
}

.filters input {
    flex: 1;
    min-width: 180px;
}

.filters input:focus,
.filters select:focus {
    border-color: var(--accent);
}

button, .button {
    display: inline-flex;
    align-items: center;
    justify-content: center;
    border: 0;
    border-radius: 8px;
    padding: 10px 14px;
    font-size: 12px;
    font-weight: 700;
    cursor: pointer;
    text-decoration: none;
    transition: .15s;
}

.primary { background: var(--accent); color: white; }
.primary:hover { background: #829dff; }

.secondary { background: #263449; color: var(--text); }
.secondary:hover { background: #344660; }

.danger { background: #49232e; color: #ff9aaa; }
.success { background: #123c31; color: #71e7b7; }
.warning { background: #49391d; color: #f5d18a; }

.table-wrap { overflow-x: auto; }

table {
    width: 100%;
    border-collapse: collapse;
    min-width: 1050px;
}

th {
    padding: 15px 17px;
    text-align: left;
    color: #8191a8;
    background: #111b2b;
    font-size: 10px;
    letter-spacing: 1px;
    text-transform: uppercase;
}

td {
    padding: 16px 17px;
    border-top: 1px solid #202c3e;
    font-size: 12px;
    white-space: nowrap;
}

tr:hover td { background: rgba(108,140,255,.035); }

.order-id { font-weight: 750; color: #dce5ff; }

.client-name { font-weight: 700; font-size: 12px; }

.client-email {
    color: var(--muted);
    font-size: 11px;
    margin-top: 5px;
}

.plan-name { font-weight: 650; }

.amount { font-weight: 750; color: #dce5ff; }

.status {
    display: inline-block;
    padding: 6px 10px;
    border-radius: 20px;
    font-size: 10px;
    font-weight: 700;
}

.status.pending { color: var(--yellow); background: #49391d; }
.status.paid { color: var(--green); background: #123329; }
.status.cancelled { color: #c1c9d6; background: #293345; }
.status.failed { color: var(--red); background: #3b2029; }

.actions {
    display: flex;
    align-items: center;
    gap: 6px;
}

.actions form { display: inline; }
.actions button { padding: 8px 10px; font-size: 10px; }

.status-form {
    display: flex;
    align-items: center;
    gap: 6px;
}

.status-form select {
    max-width: 112px;
    padding: 8px;
    border: 1px solid var(--border);
    border-radius: 7px;
    background: #0b1321;
    color: var(--text);
    font-size: 10px;
}

.status-form button {
    padding: 8px 10px;
    font-size: 10px;
}

.detail-grid {
    display: grid;
    grid-template-columns: repeat(3, minmax(0, 1fr));
    gap: 15px;
    padding: 22px;
}

.detail-item {
    padding: 15px;
    border: 1px solid var(--border);
    border-radius: 9px;
    background: #0d1625;
    min-width: 0;
}

.detail-label {
    color: var(--muted);
    font-size: 10px;
    text-transform: uppercase;
    letter-spacing: .8px;
}

.detail-value {
    margin-top: 8px;
    font-size: 13px;
    font-weight: 650;
    overflow-wrap: anywhere;
}

.pagination {
    display: flex;
    align-items: center;
    justify-content: space-between;
    gap: 12px;
    padding: 17px 22px;
    border-top: 1px solid var(--border);
    color: var(--muted);
    font-size: 11px;
    flex-wrap: wrap;
}

.pagination-links {
    display: flex;
    gap: 7px;
}

.pagination-links a {
    padding: 8px 11px;
    border: 1px solid var(--border);
    border-radius: 7px;
    color: var(--text);
    text-decoration: none;
}

.pagination-links a.current {
    background: #26385d;
    border-color: var(--accent);
}

.empty {
    padding: 35px;
    text-align: center;
    color: var(--muted);
}

.footer {
    color: #66758a;
    font-size: 11px;
    margin-top: 22px;
    text-align: right;
}

@media (max-width: 1100px) {
    .stats { grid-template-columns: repeat(2, minmax(0, 1fr)); }
    .detail-grid { grid-template-columns: repeat(2, minmax(0, 1fr)); }
}

@media (max-width: 900px) {
    .sidebar { width: 190px; }
    .main { padding: 25px 18px; }
}

@media (max-width: 650px) {
    .sidebar { display: none; }
    .stats, .detail-grid { grid-template-columns: 1fr; }
    .topbar { align-items: flex-start; flex-direction: column; }
    .toolbar { align-items: stretch; }
}
</style>
</head>

<body>
<div class="layout">

<aside class="sidebar">
    <div class="brand">Hosting<span>Lab</span></div>
    <div class="brand-subtitle">Administration</div>

    <div class="nav-label">Overview</div>
    <a class="nav-link" href="index.php">Dashboard</a>

    <div class="nav-label">Management</div>
    <a class="nav-link" href="plans.php">Hosting Plans</a>
    <a class="nav-link" href="services.php">Services</a>
    <a class="nav-link active" href="orders.php">Orders</a>
    <a class="nav-link" href="clients.php">Clients</a>

    <div class="nav-label">System</div>
    <a class="nav-link" href="logs.php">Activity Logs</a>
    <a class="nav-link" href="settings.php">Settings</a>
</aside>

<main class="main">

    <div class="topbar">
        <div>
            <div class="breadcrumb">Administration / Orders</div>
            <h1>Orders Management</h1>
            <div class="subtitle">Review customer orders and manage their current status.</div>
        </div>

        <div class="environment">LOCAL ENVIRONMENT</div>
    </div>

    <?php if (is_array($flash)): ?>
        <div class="flash <?= ($flash['type'] ?? '') === 'error' ? 'error' : 'success' ?>">
            <?= adminEscape($flash['message'] ?? '') ?>
        </div>
    <?php endif; ?>

    <section class="stats">
        <div class="stat-card">
            <div class="stat-label">Total Orders</div>
            <div class="stat-value"><?= number_format((int)($stats['total'] ?? 0)) ?></div>
        </div>

        <div class="stat-card">
            <div class="stat-label">Pending Orders</div>
            <div class="stat-value yellow"><?= number_format((int)($stats['pending_count'] ?? 0)) ?></div>
        </div>

        <div class="stat-card">
            <div class="stat-label">Paid Orders</div>
            <div class="stat-value green"><?= number_format((int)($stats['paid_count'] ?? 0)) ?></div>
        </div>

        <div class="stat-card">
            <div class="stat-label">Recorded Revenue</div>
            <div class="stat-value green">â‚¬<?= number_format((float)($stats['revenue'] ?? 0), 2) ?></div>
        </div>
    </section>

    <section class="panel">
        <div class="panel-header">
            <h2>Order Records</h2>
            <p>Orders are loaded directly from the Hosting Lab database.</p>
        </div>

        <div class="notice">
            Changing an order to Paid here is a manual administrative status update.
            It does not verify a payment or create a hosting service.
        </div>

        <div class="toolbar">
            <form class="filters" method="get" action="orders.php">
                <input
                    type="search"
                    name="q"
                    value="<?= adminEscape($search) ?>"
                    placeholder="Search order ID, client, email or plan..."
                >

                <select name="status">
                    <option value="all" <?= $statusFilter === 'all' ? 'selected' : '' ?>>All statuses</option>
                    <option value="pending" <?= $statusFilter === 'pending' ? 'selected' : '' ?>>Pending</option>
                    <option value="paid" <?= $statusFilter === 'paid' ? 'selected' : '' ?>>Paid</option>
                    <option value="cancelled" <?= $statusFilter === 'cancelled' ? 'selected' : '' ?>>Cancelled</option>
                    <option value="failed" <?= $statusFilter === 'failed' ? 'selected' : '' ?>>Failed</option>
                </select>

                <button type="submit" class="primary">Apply Filters</button>

                <?php if ($search !== '' || $statusFilter !== 'all'): ?>
                    <a class="button secondary" href="orders.php">Clear</a>
                <?php endif; ?>
            </form>

            <div class="stat-label">
                <?= number_format($totalOrders) ?> matching order(s)
            </div>
        </div>

        <?php if (!$orders): ?>
            <div class="empty">
                <?= $search !== '' || $statusFilter !== 'all'
                    ? 'No orders match the selected filters.'
                    : 'No orders have been created yet.' ?>
            </div>
        <?php else: ?>
            <div class="table-wrap">
                <table>
                    <thead>
                        <tr>
                            <th>Order</th>
                            <th>Client</th>
                            <th>Hosting Plan</th>
                            <th>Amount</th>
                            <th>Status</th>
                            <th>Created</th>
                            <th>Actions</th>
                        </tr>
                    </thead>

                    <tbody>
                    <?php foreach ($orders as $order): ?>
                        <tr>
                            <td>
                                <span class="order-id">#<?= (int)$order['id'] ?></span>
                            </td>

                            <td>
                                <div class="client-name"><?= adminEscape($order['client_name']) ?></div>
                                <div class="client-email"><?= adminEscape($order['client_email']) ?></div>
                            </td>

                            <td>
                                <div class="plan-name"><?= adminEscape($order['plan_name']) ?></div>
                                <div class="client-email"><?= adminEscape(ucfirst($order['billing_cycle'])) ?></div>
                            </td>

                            <td>
                                <span class="amount">â‚¬<?= number_format((float)$order['amount'], 2) ?></span>
                            </td>

                            <td>
                                <span class="status <?= adminEscape($order['status']) ?>">
                                    <?= adminEscape(orderStatusLabel($order['status'])) ?>
                                </span>
                            </td>

                            <td>
                                <?= adminEscape(date('d M Y H:i', strtotime($order['created_at']))) ?>
                            </td>

                            <td>
                                <div class="actions">
                                    <a
                                        class="button secondary"
                                        href="orders.php?view=<?= (int)$order['id'] ?>"
                                    >View</a>

                                    <form
                                        class="status-form"
                                        method="post"
                                        action="orders.php"
                                        onsubmit="return confirm('Change the status of order #<?= (int)$order['id'] ?>?')"
                                    >
                                        <input type="hidden" name="csrf_token" value="<?= adminEscape($_SESSION['admin_csrf']) ?>">
                                        <input type="hidden" name="action" value="update_status">
                                        <input type="hidden" name="order_id" value="<?= (int)$order['id'] ?>">

                                        <select name="status" aria-label="New order status">
                                            <?php foreach ($allowedStatuses as $status): ?>
                                                <option
                                                    value="<?= adminEscape($status) ?>"
                                                    <?= $order['status'] === $status ? 'selected' : '' ?>
                                                >
                                                    <?= adminEscape(orderStatusLabel($status)) ?>
                                                </option>
                                            <?php endforeach; ?>
                                        </select>

                                        <button type="submit" class="primary">Save</button>
                                    </form>
                                </div>
                            </td>
                        </tr>
                    <?php endforeach; ?>
                    </tbody>
                </table>
            </div>

            <div class="pagination">
                <div>Page <?= $page ?> of <?= $totalPages ?></div>

                <div class="pagination-links">
                    <?php if ($page > 1): ?>
                        <a href="<?= adminEscape(ordersPageUrl($page - 1, $search, $statusFilter)) ?>">Previous</a>
                    <?php endif; ?>

                    <?php
                    $startPage = max(1, $page - 2);
                    $endPage = min($totalPages, $page + 2);
                    ?>

                    <?php for ($pageNumber = $startPage; $pageNumber <= $endPage; $pageNumber++): ?>
                        <a
                            class="<?= $pageNumber === $page ? 'current' : '' ?>"
                            href="<?= adminEscape(ordersPageUrl($pageNumber, $search, $statusFilter)) ?>"
                        ><?= $pageNumber ?></a>
                    <?php endfor; ?>

                    <?php if ($page < $totalPages): ?>
                        <a href="<?= adminEscape(ordersPageUrl($page + 1, $search, $statusFilter)) ?>">Next</a>
                    <?php endif; ?>
                </div>
            </div>
        <?php endif; ?>
    </section>

    <?php if ($viewId): ?>
        <section class="panel">
            <div class="panel-header">
                <h2>Order Details</h2>
                <p>Full information for the selected order.</p>
            </div>

            <?php if (!$selectedOrder): ?>
                <div class="empty">The selected order could not be found.</div>
            <?php else: ?>
                <div class="detail-grid">
                    <div class="detail-item">
                        <div class="detail-label">Order ID</div>
                        <div class="detail-value">#<?= (int)$selectedOrder['id'] ?></div>
                    </div>

                    <div class="detail-item">
                        <div class="detail-label">Order Amount</div>
                        <div class="detail-value">â‚¬<?= number_format((float)$selectedOrder['amount'], 2) ?></div>
                    </div>

                    <div class="detail-item">
                        <div class="detail-label">Order Status</div>
                        <div class="detail-value">
                            <span class="status <?= adminEscape($selectedOrder['status']) ?>">
                                <?= adminEscape(orderStatusLabel($selectedOrder['status'])) ?>
                            </span>
                        </div>
                    </div>

                    <div class="detail-item">
                        <div class="detail-label">Client Name</div>
                        <div class="detail-value"><?= adminEscape($selectedOrder['client_name']) ?></div>
                    </div>

                    <div class="detail-item">
                        <div class="detail-label">Client Email</div>
                        <div class="detail-value"><?= adminEscape($selectedOrder['client_email']) ?></div>
                    </div>

                    <div class="detail-item">
                        <div class="detail-label">Client Account Status</div>
                        <div class="detail-value"><?= adminEscape(ucfirst($selectedOrder['client_status'])) ?></div>
                    </div>

                    <div class="detail-item">
                        <div class="detail-label">Hosting Plan</div>
                        <div class="detail-value"><?= adminEscape($selectedOrder['plan_name']) ?></div>
                    </div>

                    <div class="detail-item">
                        <div class="detail-label">Plan Billing Cycle</div>
                        <div class="detail-value"><?= adminEscape(ucfirst($selectedOrder['billing_cycle'])) ?></div>
                    </div>

                    <div class="detail-item">
                        <div class="detail-label">Plan Listed Price</div>
                        <div class="detail-value">â‚¬<?= number_format((float)$selectedOrder['plan_price'], 2) ?></div>
                    </div>

                    <div class="detail-item">
                        <div class="detail-label">Client ID</div>
                        <div class="detail-value">#<?= (int)$selectedOrder['user_id'] ?></div>
                    </div>

                    <div class="detail-item">
                        <div class="detail-label">Plan ID</div>
                        <div class="detail-value">#<?= (int)$selectedOrder['plan_id'] ?></div>
                    </div>

                    <div class="detail-item">
                        <div class="detail-label">Created At</div>
                        <div class="detail-value"><?= adminEscape(date('d M Y H:i:s', strtotime($selectedOrder['created_at']))) ?></div>
                    </div>

                    <div class="detail-item">
                        <div class="detail-label">SSD Storage</div>
                        <div class="detail-value"><?= number_format((int)$selectedOrder['disk_limit_mb'] / 1024, 2) ?> GB</div>
                    </div>

                    <div class="detail-item">
                        <div class="detail-label">Bandwidth</div>
                        <div class="detail-value">
                            <?= (int)$selectedOrder['bandwidth_limit_mb'] === 0
                                ? 'Unlimited'
                                : number_format((int)$selectedOrder['bandwidth_limit_mb'] / 1024, 2) . ' GB' ?>
                        </div>
                    </div>

                    <div class="detail-item">
                        <div class="detail-label">Domains</div>
                        <div class="detail-value"><?= number_format((int)$selectedOrder['domains_limit']) ?></div>
                    </div>

                    <div class="detail-item">
                        <div class="detail-label">Databases</div>
                        <div class="detail-value">
                            <?= (int)$selectedOrder['databases_limit'] >= 400
                                ? 'Unlimited'
                                : number_format((int)$selectedOrder['databases_limit']) ?>
                        </div>
                    </div>

                    <div class="detail-item">
                        <div class="detail-label">FTP Accounts</div>
                        <div class="detail-value"><?= number_format((int)$selectedOrder['ftp_accounts_limit']) ?></div>
                    </div>
                </div>

                <?php if (!empty($selectedOrder['plan_description'])): ?>
                    <div class="notice">
                        <?= nl2br(adminEscape($selectedOrder['plan_description'])) ?>
                    </div>
                <?php endif; ?>

                <div style="padding: 0 22px 22px;">
                    <a class="button secondary" href="orders.php">Close Details</a>
                </div>
            <?php endif; ?>
        </section>
    <?php endif; ?>

    <div class="footer">
        Hosting Lab Â· Orders Management
    </div>

</main>
</div>
</body>
</html>
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
PS D:\Program Files\XAMPP\htdocs\hosting>
