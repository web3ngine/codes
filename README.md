===== COMPLETE ADMIN SETTINGS =====
<?php
declare(strict_types=1);

require_once __DIR__ . '/../config/database.php';
require_once __DIR__ . '/../includes/auth.php';

requireAdmin($pdo);

$settingsFile = __DIR__ . '/../storage/settings.php';

$defaults = [
    'site_name' => 'Hosting Lab',
    'site_url' => 'https://hosting.web-engine.cc.cd',
    'company_name' => '',
    'support_email' => '',
    'contact_phone' => '',
    'currency' => 'EUR',
    'timezone' => 'Europe/Sofia',
    'maintenance_mode' => false,
    'maintenance_message' => 'We are currently performing scheduled maintenance. Please check back soon.',
];

function settingsEscape(mixed $value): string
{
    return htmlspecialchars((string)$value, ENT_QUOTES, 'UTF-8');
}

function settingsLoad(string $file, array $defaults): array
{
    if (!is_file($file)) {
        return $defaults;
    }

    if (!defined('HOSTING_LAB_SETTINGS_ACCESS')) {
        define('HOSTING_LAB_SETTINGS_ACCESS', true);
    }

    $loaded = require $file;

    if (!is_array($loaded)) {
        throw new RuntimeException('Settings storage has an invalid format.');
    }

    return array_merge($defaults, $loaded);
}

function settingsSave(string $file, array $settings): void
{
    $directory = dirname($file);

    if (!is_dir($directory) || !is_writable($directory)) {
        throw new RuntimeException('Settings storage directory is not writable.');
    }

    $content = "<?php\n"
        . "declare(strict_types=1);\n"
        . "if (!defined('HOSTING_LAB_SETTINGS_ACCESS')) { http_response_code(404); exit; }\n"
        . "return " . var_export($settings, true) . ";\n";

    $temporary = $file . '.' . bin2hex(random_bytes(8)) . '.tmp';

    if (file_put_contents($temporary, $content, LOCK_EX) === false) {
        throw new RuntimeException('Could not write temporary settings file.');
    }

    if (!@rename($temporary, $file)) {
        @unlink($temporary);
        throw new RuntimeException('Could not replace settings file.');
    }

    @chmod($file, 0600);
}

$settings = settingsLoad($settingsFile, $defaults);
$message = '';
$messageType = 'success';

if ($_SERVER['REQUEST_METHOD'] === 'POST') {
    try {
        $token = (string)($_POST['csrf_token'] ?? '');

        if (
            $token === '' ||
            !hash_equals((string)$_SESSION['admin_csrf'], $token)
        ) {
            throw new RuntimeException('Security token is invalid. Refresh the page and try again.');
        }

        $siteName = trim((string)($_POST['site_name'] ?? ''));
        $siteUrl = trim((string)($_POST['site_url'] ?? ''));
        $companyName = trim((string)($_POST['company_name'] ?? ''));
        $supportEmail = trim((string)($_POST['support_email'] ?? ''));
        $contactPhone = trim((string)($_POST['contact_phone'] ?? ''));
        $currency = strtoupper(trim((string)($_POST['currency'] ?? 'EUR')));
        $timezone = trim((string)($_POST['timezone'] ?? 'Europe/Sofia'));
        $maintenanceMessage = trim((string)($_POST['maintenance_message'] ?? ''));

        if ($siteName === '' || mb_strlen($siteName) > 100) {
            throw new RuntimeException('Platform name is required and must be under 100 characters.');
        }

        if (
            !filter_var($siteUrl, FILTER_VALIDATE_URL) ||
            !in_array(strtolower((string)parse_url($siteUrl, PHP_URL_SCHEME)), ['http', 'https'], true)
        ) {
            throw new RuntimeException('Enter a valid HTTP or HTTPS platform URL.');
        }

        if (mb_strlen($siteUrl) > 255) {
            throw new RuntimeException('Platform URL is too long.');
        }

        if ($companyName !== '' && mb_strlen($companyName) > 150) {
            throw new RuntimeException('Company name is too long.');
        }

        if (
            $supportEmail !== '' &&
            (!filter_var($supportEmail, FILTER_VALIDATE_EMAIL) || mb_strlen($supportEmail) > 190)
        ) {
            throw new RuntimeException('Enter a valid support email address.');
        }

        if (mb_strlen($contactPhone) > 40) {
            throw new RuntimeException('Phone number is too long.');
        }

        if (!in_array($currency, ['EUR', 'USD', 'GBP'], true)) {
            throw new RuntimeException('Unsupported currency.');
        }

        if (!in_array($timezone, DateTimeZone::listIdentifiers(), true)) {
            throw new RuntimeException('Select a valid timezone identifier.');
        }

        if ($maintenanceMessage === '' || mb_strlen($maintenanceMessage) > 500) {
            throw new RuntimeException('Maintenance message is required and must be under 500 characters.');
        }

        $updated = [
            'site_name' => $siteName,
            'site_url' => rtrim($siteUrl, '/'),
            'company_name' => $companyName,
            'support_email' => $supportEmail,
            'contact_phone' => $contactPhone,
            'currency' => $currency,
            'timezone' => $timezone,
            'maintenance_mode' => isset($_POST['maintenance_mode']),
            'maintenance_message' => $maintenanceMessage,
        ];

        settingsSave($settingsFile, $updated);
        $settings = $updated;

        $adminId = (int)$_SESSION['admin_id'];
        $logStmt = $pdo->prepare(
            "INSERT INTO activity_logs (user_id, action, details, ip_address)
             VALUES (:user_id, :action, :details, :ip)"
        );

        $logStmt->execute([
            ':user_id' => $adminId,
            ':action' => 'settings.updated',
            ':details' => 'Platform settings were updated.',
            ':ip' => substr((string)($_SERVER['REMOTE_ADDR'] ?? ''), 0, 45),
        ]);

        $_SESSION['admin_csrf'] = bin2hex(random_bytes(32));
        $message = 'Settings saved successfully.';
    } catch (Throwable $e) {
        error_log('Hosting Lab settings error: ' . $e->getMessage());
        $message = $e instanceof RuntimeException
            ? $e->getMessage()
            : 'An unexpected error occurred while saving settings.';
        $messageType = 'error';
    }
}

$adminName = $_SESSION['admin_name'] ?? 'Administrator';
$timezones = ['Europe/Sofia', 'Europe/London', 'Europe/Berlin', 'Europe/Paris', 'UTC', 'America/New_York', 'America/Los_Angeles'];
?>
<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Settings | Hosting Lab</title>
<style>
:root {
    --bg:#090d16;
    --panel:#111827;
    --panel2:#151f30;
    --border:#202c40;
    --text:#e8edf7;
    --muted:#8b9bb4;
    --accent:#6c8cff;
    --green:#35c98b;
    --red:#f06b78;
}
*{box-sizing:border-box}
body{margin:0;background:var(--bg);color:var(--text);font-family:Inter,Segoe UI,Arial,sans-serif}
.layout{display:flex;min-height:100vh}
.sidebar{width:245px;background:#0d1421;border-right:1px solid var(--border);padding:25px 17px;flex-shrink:0}
.brand{font-size:21px;font-weight:800;margin:0 0 35px;padding:0 10px}
.brand span{color:var(--accent)}
.nav-label{font-size:10px;letter-spacing:1.5px;color:#61718c;padding:18px 10px 9px;text-transform:uppercase}
.nav-link{display:block;color:#9cabc2;text-decoration:none;padding:11px 12px;border-radius:9px;margin:3px 0;font-size:13px}
.nav-link:hover,.nav-link.active{background:#1b2941;color:white}
.nav-link.active{border-left:3px solid var(--accent)}
.main{flex:1;min-width:0;padding:30px 36px}
.topbar{display:flex;justify-content:space-between;align-items:center;gap:15px;margin-bottom:30px}
.topbar h1{font-size:25px;margin:0}
.topbar p{font-size:12px;color:var(--muted);margin:7px 0 0}
.admin{font-size:12px;color:var(--muted)}
.content{max-width:1050px}
.card{background:var(--panel);border:1px solid var(--border);border-radius:13px;padding:24px;margin-bottom:20px}
.card-head{margin-bottom:22px}
.card-head h2{font-size:16px;margin:0 0 6px}
.card-head p{font-size:12px;color:var(--muted);margin:0}
.grid{display:grid;grid-template-columns:1fr 1fr;gap:18px}
.field{display:flex;flex-direction:column;gap:8px}
.field.full{grid-column:1/-1}
label{font-size:12px;color:#b9c6da;font-weight:600}
input,select,textarea{width:100%;background:#0b1220;color:var(--text);border:1px solid #29364d;border-radius:8px;padding:12px;font:inherit;font-size:13px;outline:none}
input:focus,select:focus,textarea:focus{border-color:var(--accent)}
textarea{min-height:95px;resize:vertical}
.hint{font-size:11px;color:var(--muted)}
.toggle-row{display:flex;justify-content:space-between;align-items:center;gap:15px;background:var(--panel2);border:1px solid var(--border);padding:16px;border-radius:10px}
.toggle-row strong{font-size:13px}
.toggle-row p{font-size:11px;color:var(--muted);margin:5px 0 0}
.toggle-row input{width:18px;height:18px;accent-color:var(--accent)}
.alert{padding:13px 15px;border-radius:9px;margin-bottom:18px;font-size:13px}
.alert.success{background:#123326;color:#7ce0b2;border:1px solid #205a43}
.alert.error{background:#3b1d28;color:#ff9ba5;border:1px solid #713444}
.actions{display:flex;justify-content:flex-end;margin-top:20px}
button{background:var(--accent);color:white;border:0;border-radius:8px;padding:12px 22px;font-size:13px;font-weight:700;cursor:pointer}
button:hover{filter:brightness(1.1)}
.footer-note{font-size:11px;color:var(--muted);margin-top:16px}
@media(max-width:850px){.sidebar{width:200px}.main{padding:25px 20px}}
@media(max-width:650px){.layout{display:block}.sidebar{width:100%;padding:15px}.brand{margin-bottom:12px}.nav-label{padding-top:10px}.main{padding:20px 14px}.grid{grid-template-columns:1fr}.field.full{grid-column:auto}.topbar{align-items:flex-start;flex-direction:column}}
</style>
</head>
<body>
<div class="layout">
<aside class="sidebar">
    <div class="brand">Hosting <span>Lab</span></div>
    <div class="nav-label">Overview</div>
    <a class="nav-link" href="index.php">Dashboard</a>
    <div class="nav-label">Management</div>
    <a class="nav-link" href="plans.php">Hosting Plans</a>
    <a class="nav-link" href="services.php">Services</a>
    <a class="nav-link" href="orders.php">Orders</a>
    <a class="nav-link" href="clients.php">Clients</a>
    <div class="nav-label">System</div>
    <a class="nav-link" href="logs.php">Activity Logs</a>
    <a class="nav-link active" href="settings.php">Settings</a>
</aside>
<main class="main">
    <div class="topbar">
        <div>
            <h1>Platform Settings</h1>
            <p>Manage your Hosting Lab platform configuration.</p>
        </div>
        <div class="admin">Signed in as <?= settingsEscape($adminName) ?></div>
    </div>

    <div class="content">
        <?php if ($message !== ''): ?>
            <div class="alert <?= settingsEscape($messageType) ?>"><?= settingsEscape($message) ?></div>
        <?php endif; ?>

        <form method="post" action="settings.php">
            <input type="hidden" name="csrf_token" value="<?= settingsEscape($_SESSION['admin_csrf']) ?>">

            <section class="card">
                <div class="card-head">
                    <h2>General Information</h2>
                    <p>Basic identity and contact details for your hosting platform.</p>
                </div>
                <div class="grid">
                    <div class="field">
                        <label for="site_name">Platform Name</label>
                        <input id="site_name" name="site_name" maxlength="100" required value="<?= settingsEscape($settings['site_name']) ?>">
                    </div>
                    <div class="field">
                        <label for="site_url">Platform URL</label>
                        <input id="site_url" name="site_url" type="url" maxlength="255" required value="<?= settingsEscape($settings['site_url']) ?>">
                    </div>
                    <div class="field">
                        <label for="company_name">Company Name</label>
                        <input id="company_name" name="company_name" maxlength="150" value="<?= settingsEscape($settings['company_name']) ?>">
                    </div>
                    <div class="field">
                        <label for="support_email">Support Email</label>
                        <input id="support_email" name="support_email" type="email" maxlength="190" value="<?= settingsEscape($settings['support_email']) ?>">
                    </div>
                    <div class="field">
                        <label for="contact_phone">Contact Phone</label>
                        <input id="contact_phone" name="contact_phone" maxlength="40" value="<?= settingsEscape($settings['contact_phone']) ?>">
                    </div>
                </div>
            </section>

            <section class="card">
                <div class="card-head">
                    <h2>Regional Settings</h2>
                    <p>Currency and timezone used by the platform.</p>
                </div>
                <div class="grid">
                    <div class="field">
                        <label for="currency">Default Currency</label>
                        <select id="currency" name="currency">
                            <?php foreach (['EUR' => 'Euro (EUR)', 'USD' => 'US Dollar (USD)', 'GBP' => 'British Pound (GBP)'] as $code => $label): ?>
                                <option value="<?= settingsEscape($code) ?>" <?= $settings['currency'] === $code ? 'selected' : '' ?>><?= settingsEscape($label) ?></option>
                            <?php endforeach; ?>
                        </select>
                    </div>
                    <div class="field">
                        <label for="timezone">Timezone</label>
                        <select id="timezone" name="timezone">
                            <?php foreach ($timezones as $zone): ?>
                                <option value="<?= settingsEscape($zone) ?>" <?= $settings['timezone'] === $zone ? 'selected' : '' ?>><?= settingsEscape($zone) ?></option>
                            <?php endforeach; ?>
                        </select>
                    </div>
                </div>
            </section>

            <section class="card">
                <div class="card-head">
                    <h2>Maintenance Mode</h2>
                    <p>Store the maintenance state and message for future integration with the client-facing pages.</p>
                </div>
                <div class="toggle-row">
                    <div>
                        <strong>Enable Maintenance Mode</strong>
                        <p>This saves the setting. It does not yet block public pages.</p>
                    </div>
                    <input type="checkbox" name="maintenance_mode" value="1" <?= !empty($settings['maintenance_mode']) ? 'checked' : '' ?>>
                </div>
                <div class="field" style="margin-top:18px">
                    <label for="maintenance_message">Maintenance Message</label>
                    <textarea id="maintenance_message" name="maintenance_message" maxlength="500" required><?= settingsEscape($settings['maintenance_message']) ?></textarea>
                    <span class="hint">Maximum 500 characters.</span>
                </div>
            </section>

            <div class="actions">
                <button type="submit">Save Settings</button>
            </div>
        </form>
        <p class="footer-note">Settings are stored locally in a protected PHP file. Database schema is not modified.</p>
    </div>
</main>
</div>
</body>
</html>

===== CURRENT STORAGE SETTINGS =====
<?php
declare(strict_types=1);

if (!defined('HOSTING_LAB_SETTINGS_ACCESS')) {
    http_response_code(404);
    exit;
}

return [
    'site_name' => 'Hosting Lab',
    'site_url' => 'https://hosting.web-engine.cc.cd',
    'company_name' => '',
    'support_email' => '',
    'contact_phone' => '',
    'currency' => 'EUR',
    'timezone' => 'Europe/Sofia',
    'maintenance_mode' => false,
    'maintenance_message' => 'We are currently performing scheduled maintenance. Please check back soon.',
];

===== HOSTING STORAGE MANAGER =====
<?php

declare(strict_types=1);

namespace WebEngine\Hosting;

final class StorageManager
{
    public const ROOT = 'D:\\HostingData';

    private const DIRECTORIES = [
        'accounts',
        'ftp',
        'backups',
        'database-backups',
        'logs',
        'system',
        'temp',
    ];

    private function __construct()
    {
    }

    public static function healthCheck(): array
    {
        $results = [];

        $paths = ['root' => self::ROOT];

        foreach (self::DIRECTORIES as $directory) {
            $paths[$directory] = self::ROOT . DIRECTORY_SEPARATOR . $directory;
        }

        foreach ($paths as $name => $path) {
            $exists = is_dir($path);

            $results[$name] = [
                'path' => $path,
                'exists' => $exists,
                'writable' => $exists && is_writable($path),
            ];
        }

        return $results;
    }

    public static function accountPath(string $username): string
    {
        if (!preg_match('/\Anh[a-f0-9]{12}\z/', $username)) {
            throw new \InvalidArgumentException('Invalid hosting account username.');
        }

        return self::ROOT . DIRECTORY_SEPARATOR
            . 'accounts' . DIRECTORY_SEPARATOR . $username;
    }

    public static function ftpPath(string $username): string
    {
        if (!preg_match('/\Anh[a-f0-9]{12}\z/', $username)) {
            throw new \InvalidArgumentException('Invalid hosting account username.');
        }

        return self::ROOT . DIRECTORY_SEPARATOR
            . 'ftp' . DIRECTORY_SEPARATOR . $username;
    }
}
