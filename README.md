===== PLUGIN JSON CONTENT =====

--- effect-c4-bomb ---
{
    "name": "CS 1.6 C4 Bomb",
    "version": "1.0.0",
    "author": "WebEngine",
    "description": "CS 1.6 C4 bomb (3D image with blinking LED)",
    "main": "index.php",
    "icon": "fa-bomb",
    "category": "fun"
}

--- effect-christmas-bg ---
{
    "name": "Christmas Background",
    "version": "1.0.0",
    "author": "WebEngine",
    "description": "Replace site background with a 4K Christmas image (presets + custom URL)",
    "main": "index.php",
    "icon": "fa-image",
    "category": "christmas"
}

--- effect-christmas-paws ---
{
    "name": "Christmas Paws",
    "version": "1.0.0",
    "author": "WebEngine",
    "description": "3D paw prints follow the mouse cursor",
    "main": "index.php",
    "icon": "fa-paw",
    "category": "christmas"
}

--- effect-christmas-tree ---
{
    "name": "Christmas Tree",
    "version": "1.0.0",
    "author": "WebEngine",
    "description": "3D Christmas tree in bottom-left corner",
    "main": "index.php",
    "icon": "fa-tree",
    "category": "christmas"
}

--- effect-gifts ---
{
    "name": "Falling Gifts",
    "version": "1.0.0",
    "author": "WebEngine",
    "description": "3D Christmas gifts falling from the sky",
    "main": "index.php",
    "icon": "fa-gift",
    "category": "christmas"
}

--- effect-leaves ---
{
    "name": "Falling Leaves",
    "version": "1.0.0",
    "author": "WebEngine",
    "description": "3D autumn leaves falling (multiple image URLs)",
    "main": "index.php",
    "icon": "fa-leaf",
    "category": "autumn"
}

--- effect-lightning ---
{
    "name": "Lightning Effect",
    "version": "1.0.0",
    "author": "WebEngine",
    "description": "Random lightning flashes across the screen",
    "main": "index.php",
    "icon": "fa-bolt",
    "category": "weather"
}

--- effect-rain ---
{
    "name": "Rain Effect",
    "version": "1.0.0",
    "author": "WebEngine",
    "description": "Animated falling rain drops",
    "main": "index.php",
    "icon": "fa-cloud-rain",
    "category": "weather"
}

--- effect-santa-hat ---
{
    "name": "Santa Hat on Logo",
    "version": "1.0.0",
    "author": "WebEngine",
    "description": "3D Santa hat on the site logo",
    "main": "index.php",
    "icon": "fa-hat-wizard",
    "category": "christmas"
}

--- effect-santa-sled ---
{
    "name": "Santa Sled",
    "version": "1.0.0",
    "author": "WebEngine",
    "description": "3D Santa on a sled travelling across the site",
    "main": "index.php",
    "icon": "fa-sleigh",
    "category": "christmas"
}

--- effect-snow ---
{
    "name": "Snow Effect",
    "version": "1.0.0",
    "author": "WebEngine",
    "description": "Animated falling snowflakes (3D image support)",
    "main": "index.php",
    "icon": "fa-snowflake",
    "category": "christmas"
}

--- effect-train ---
{
    "name": "Christmas Train",
    "version": "1.0.0",
    "author": "WebEngine",
    "description": "3D train travels along the footer",
    "main": "index.php",
    "icon": "fa-train",
    "category": "christmas"
}

--- effect-witch ---
{
    "name": "Witch on Broom",
    "version": "1.0.0",
    "author": "WebEngine",
    "description": "3D witch flies across the site on a broom",
    "main": "index.php",
    "icon": "fa-hat-wizard",
    "category": "halloween"
}

===== PLUGIN HOOKS SOURCE =====
<?php
declare(strict_types=1);

/* ==========================================================
   WEBENGINE â€” PLUGIN HOOKS SYSTEM
   File: webengine/includes/plugin_hooks.php
   Purpose: Simple WordPress-like hooks (actions + shortcodes)
            for plugins. Loaded once via header.php.
   ========================================================== */

if (!defined('WE_PLUGIN_HOOKS_LOADED')) {
    define('WE_PLUGIN_HOOKS_LOADED', true);

    /* ==========================================================
       INTERNAL STORAGE
       ========================================================== */

    if (!isset($GLOBALS['we_actions'])) {
        $GLOBALS['we_actions'] = [];
    }

    if (!isset($GLOBALS['we_shortcodes'])) {
        $GLOBALS['we_shortcodes'] = [];
    }

    if (!isset($GLOBALS['we_filters'])) {
        $GLOBALS['we_filters'] = [];
    }

    /* ==========================================================
       ACTION HOOKS
       add_action('head', function() { ... });
       do_action('head');
       ========================================================== */

    if (!function_exists('add_action')) {
        function add_action(string $hook, callable $callback, int $priority = 10): void {
            $hook = preg_replace('/[^a-z0-9_\-]/i', '', $hook);
            if ($hook === '') return;

            if (!isset($GLOBALS['we_actions'][$hook])) {
                $GLOBALS['we_actions'][$hook] = [];
            }
            if (!isset($GLOBALS['we_actions'][$hook][$priority])) {
                $GLOBALS['we_actions'][$hook][$priority] = [];
            }

            $GLOBALS['we_actions'][$hook][$priority][] = $callback;
        }
    }

    if (!function_exists('do_action')) {
        function do_action(string $hook, ...$args): void {
            $hook = preg_replace('/[^a-z0-9_\-]/i', '', $hook);
            if ($hook === '') return;

            if (empty($GLOBALS['we_actions'][$hook])) return;

            $callbacks = $GLOBALS['we_actions'][$hook];
            ksort($callbacks);

            foreach ($callbacks as $priorityList) {
                foreach ($priorityList as $cb) {
                    try {
                        call_user_func_array($cb, $args);
                    } catch (Throwable $e) {
                        /* Silent â€” never break the page because of a plugin */
                    }
                }
            }
        }
    }

    if (!function_exists('remove_action')) {
        function remove_action(string $hook, callable $callback): bool {
            $hook = preg_replace('/[^a-z0-9_\-]/i', '', $hook);
            if ($hook === '' || empty($GLOBALS['we_actions'][$hook])) return false;

            foreach ($GLOBALS['we_actions'][$hook] as $priority => $list) {
                foreach ($list as $i => $cb) {
                    if ($cb === $callback) {
                        unset($GLOBALS['we_actions'][$hook][$priority][$i]);
                        return true;
                    }
                }
            }
            return false;
        }
    }

    if (!function_exists('has_action')) {
        function has_action(string $hook): bool {
            $hook = preg_replace('/[^a-z0-9_\-]/i', '', $hook);
            if ($hook === '') return false;
            return !empty($GLOBALS['we_actions'][$hook]);
        }
    }

    /* ==========================================================
       FILTER HOOKS
       add_filter('content', function($text) { return $text; });
       $text = apply_filters('content', $text);
       ========================================================== */

    if (!function_exists('add_filter')) {
        function add_filter(string $hook, callable $callback, int $priority = 10): void {
            $hook = preg_replace('/[^a-z0-9_\-]/i', '', $hook);
            if ($hook === '') return;

            if (!isset($GLOBALS['we_filters'][$hook])) {
                $GLOBALS['we_filters'][$hook] = [];
            }
            if (!isset($GLOBALS['we_filters'][$hook][$priority])) {
                $GLOBALS['we_filters'][$hook][$priority] = [];
            }

            $GLOBALS['we_filters'][$hook][$priority][] = $callback;
        }
    }

    if (!function_exists('apply_filters')) {
        function apply_filters(string $hook, $value, ...$args) {
            $hook = preg_replace('/[^a-z0-9_\-]/i', '', $hook);
            if ($hook === '') return $value;

            if (empty($GLOBALS['we_filters'][$hook])) return $value;

            $callbacks = $GLOBALS['we_filters'][$hook];
            ksort($callbacks);

            foreach ($callbacks as $priorityList) {
                foreach ($priorityList as $cb) {
                    try {
                        $value = call_user_func_array($cb, array_merge([$value], $args));
                    } catch (Throwable $e) {
                        /* Silent */
                    }
                }
            }

            return $value;
        }
    }

    /* ==========================================================
       SHORTCODES
       add_shortcode('hello', function() { return 'Hi!'; });
       echo do_shortcode('Welcome [hello]');
       ========================================================== */

    if (!function_exists('add_shortcode')) {
        function add_shortcode(string $tag, callable $callback): void {
            $tag = preg_replace('/[^a-z0-9_\-]/i', '', $tag);
            if ($tag === '') return;

            $GLOBALS['we_shortcodes'][$tag] = $callback;
        }
    }

    if (!function_exists('do_shortcode')) {
        function do_shortcode(string $content): string {
            if (empty($GLOBALS['we_shortcodes'])) return $content;

            return preg_replace_callback('/\[([a-z0-9_\-]+)([^\]]*)\]/i', function($m) {
                $tag = $m[1];
                $argsString = trim($m[2] ?? '');

                if (empty($GLOBALS['we_shortcodes'][$tag])) {
                    return $m[0];
                }

                /* Parse attributes: key="value" key2='value2' */
                $args = [];
                if ($argsString !== '') {
                    if (preg_match_all('/([a-z0-9_\-]+)\s*=\s*["\']([^"\']*)["\']/i', $argsString, $am, PREG_SET_ORDER)) {
                        foreach ($am as $pair) {
                            $args[$pair[1]] = $pair[2];
                        }
                    }
                }

                try {
                    return (string)call_user_func($GLOBALS['we_shortcodes'][$tag], $args);
                } catch (Throwable $e) {
                    return '';
                }
            }, $content) ?? $content;
        }
    }

    if (!function_exists('shortcode_exists')) {
        function shortcode_exists(string $tag): bool {
            $tag = preg_replace('/[^a-z0-9_\-]/i', '', $tag);
            if ($tag === '') return false;
            return isset($GLOBALS['we_shortcodes'][$tag]);
        }
    }

    /* ==========================================================
       DEBUG HELPERS
       ========================================================== */

    if (!function_exists('we_get_all_actions')) {
        function we_get_all_actions(): array {
            return $GLOBALS['we_actions'];
        }
    }

    if (!function_exists('we_get_all_shortcodes')) {
        function we_get_all_shortcodes(): array {
            return array_keys($GLOBALS['we_shortcodes']);
        }
    }

    /* ==========================================================
       AUTO-LOAD ACTIVE PLUGINS
       (runs once, right after hooks are registered)
       ========================================================== */

    if (isset($pdo) && $pdo instanceof PDO && !defined('WE_PLUGINS_LOADED')) {
        define('WE_PLUGINS_LOADED', true);

        try {
            /* Ensure plugins table exists */
            $pdo->exec("
                CREATE TABLE IF NOT EXISTS `plugins` (
                    `id` INT(11) NOT NULL AUTO_INCREMENT,
                    `slug` VARCHAR(100) NOT NULL UNIQUE,
                    `name` VARCHAR(255) NOT NULL,
                    `description` TEXT DEFAULT NULL,
                    `version` VARCHAR(50) DEFAULT '1.0.0',
                    `author` VARCHAR(255) DEFAULT NULL,
                    `icon` VARCHAR(100) DEFAULT 'fa-puzzle-piece',
                    `category` VARCHAR(50) DEFAULT 'general',
                    `main_file` VARCHAR(255) DEFAULT NULL,
                    `status` ENUM('active','inactive') DEFAULT 'inactive',
                    `is_required` TINYINT(1) DEFAULT 0,
                    `activated_at` DATETIME DEFAULT NULL,
                    `created_at` DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP,
                    PRIMARY KEY (`id`),
                    KEY `idx_slug` (`slug`),
                    KEY `idx_status` (`status`)
                ) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci
            ");

            /* Load all active plugins */
            $stmt = $pdo->query("
                SELECT slug, main_file
                FROM plugins
                WHERE status = 'active'
                ORDER BY id ASC
            ");

            $basePath = defined('WE_BASE_PATH') ? WE_BASE_PATH : (__DIR__ . '/..');

            while ($plugin = $stmt->fetch(PDO::FETCH_ASSOC)) {
                $slug = preg_replace('/[^a-z0-9_\-]/i', '', (string)($plugin['slug'] ?? ''));
                $main = (string)($plugin['main_file'] ?? '');

                if ($slug === '' || $main === '') continue;

                $pluginFile = rtrim($basePath, '/') . '/plugins/' . $slug . '/' . basename($main);

                if (is_file($pluginFile)) {
                    require_once $pluginFile;
                }
            }
        } catch (Throwable $e) {
            /* Silent â€” plugins table may not exist yet */
        }
    }
}

===== PLUGIN MANAGER FUNCTIONS =====

  D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:29:}
  D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:30:
> D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:31:$stmt = $pdo->prepare("SELECT id, username, avatar, role F
ROM users WHERE id = ? LIMIT 1");
  D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:32:$stmt->execute([$adminId]);
  D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:33:$admin = $stmt->fetch(PDO::FETCH_ASSOC);
  D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:34:
  D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:35:if (!$admin || $admin['role'] !== 'admin') {
  D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:91:            `icon` VARCHAR(100) DEFAULT 'fa-puzzle-piece',
  D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:92:            `category` VARCHAR(50) DEFAULT 'general',
> D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:93:            `main_file` VARCHAR(255) DEFAULT NULL,
  D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:94:            `status` ENUM('active','inactive') DEFAULT 'in
active',
  D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:95:            `is_required` TINYINT(1) DEFAULT 0,
  D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:96:            `activated_at` DATETIME DEFAULT NULL,
  D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:97:            `created_at` DATETIME NOT NULL DEFAULT CURRENT
_TIMESTAMP,
  D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:103:
  D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:104:    $pdo->exec("
> D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:105:        CREATE TABLE IF NOT EXISTS `plugin_settings` (
  D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:106:            `id` INT(11) NOT NULL AUTO_INCREMENT,
  D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:107:            `plugin_slug` VARCHAR(100) NOT NULL,
  D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:108:            `setting_key` VARCHAR(100) NOT NULL,
  D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:109:            `setting_value` TEXT DEFAULT NULL,
> D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:110:            `updated_at` DATETIME NOT NULL DEFAULT CURREN
T_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
  D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:111:            PRIMARY KEY (`id`),
  D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:112:            UNIQUE KEY `uniq_plugin_setting` (`plugin_slu
g`, `setting_key`),
  D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:113:            KEY `idx_plugin` (`plugin_slug`)
  D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:114:        ) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=u
tf8mb4_unicode_ci
  D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:120:   ======================================================
==== */
  D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:121:
> D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:122:function timeAgo(string $datetime): string {
  D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:123:    if (empty($datetime)) return '-';
  D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:124:    $diff = time() - strtotime($datetime);
  D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:125:    if ($diff < 60)       return 'just now';
  D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:126:    if ($diff < 3600)     return floor($diff / 60) . ' mi
n ago';
  D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:131:}
  D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:132:
> D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:133:function registerPlugin(PDO $pdo, array $meta, array $set
tings = []): array {
  D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:134:    $slug = $meta['slug'];
  D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:135:
  D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:136:    try {
> D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:137:        $stmt = $pdo->prepare("SELECT id FROM plugins WHE
RE slug = ? LIMIT 1");
  D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:138:        $stmt->execute([$slug]);
  D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:139:
  D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:140:        if ($stmt->fetchColumn()) {
  D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:141:            return ['status' => 'skip', 'name' => $meta['
name']];
  D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:143:
  D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:144:        $pdo->prepare("
> D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:145:            INSERT INTO plugins
> D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:146:                (slug, name, description, version, author
, icon, category, main_file, status)
  D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:147:            VALUES (?, ?, ?, ?, ?, ?, ?, ?, 'inactive')
  D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:148:        ")->execute([
  D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:149:            $slug,
  D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:150:            $meta['name'],
  D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:160:            if (is_array($v)) continue;
  D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:161:            $pdo->prepare("
> D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:162:                INSERT INTO plugin_settings (plugin_slug,
 setting_key, setting_value)
  D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:163:                VALUES (?, ?, ?)
> D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:164:                ON DUPLICATE KEY UPDATE setting_value = V
ALUES(setting_value)
  D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:165:            ")->execute([$slug, $k, (string)$v]);
  D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:166:        }
  D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:167:
  D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:168:        return ['status' => 'created', 'name' => $meta['n
ame']];
  D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:176:   ======================================================
==== */
  D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:177:
> D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:178:function getEffectPlugins(): array {
  D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:179:    $plugins = [];
  D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:180:
  D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:181:    /* ==================================================
==========
  D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:182:       1. RAIN
  D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:206:    if (isset($pdo) && $pdo instanceof PDO) {
  D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:207:        try {
> D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:208:            $__s = $pdo->prepare("SELECT setting_key,sett
ing_value FROM plugin_settings WHERE plugin_slug=?");
  D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:209:            $__s->execute(['effect-rain']);
  D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:210:            while ($__r = $__s->fetch(PDO::FETCH_ASSOC))
{ $__cfg[$__r['setting_key']] = $__r['setting_value']; }
  D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:211:        } catch (Throwable $e) {}
  D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:212:    }
  D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:268:    if (isset($pdo) && $pdo instanceof PDO) {
  D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:269:        try {
> D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:270:            $__s = $pdo->prepare("SELECT setting_key,sett
ing_value FROM plugin_settings WHERE plugin_slug=?");
  D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:271:            $__s->execute(['effect-snow']);
  D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:272:            while ($__r = $__s->fetch(PDO::FETCH_ASSOC))
{ $__cfg[$__r['setting_key']] = $__r['setting_value']; }
  D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:273:        } catch (Throwable $e) {}
  D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:274:    }
  D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:286:.we-snow-container{position:fixed;inset:0;pointer-events:
none;z-index:9998;overflow:hidden}
  D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:287:.we-snow-flake{position:absolute;top:-10%;opacity:<?= $op
acity ?>;
> D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:288:user-select:none;animation:weSnowFall linear infinite,weS
nowSway 4s ease-in-out infinite alternate}
  D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:289:.we-snow-flake img{width:100%;height:100%;display:block;f
ilter:drop-shadow(0 0 6px rgba(255,255,255,0.5))}
  D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:290:@keyframes weSnowFall{to{transform:translateY(110vh)}}
  D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:291:@keyframes weSnowSway{from{margin-left:-25px}to{margin-le
ft:25px}}
  D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:292:</style>
  D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:341:    if (isset($pdo) && $pdo instanceof PDO) {
  D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:342:        try {
> D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:343:            $__s = $pdo->prepare("SELECT setting_key,sett
ing_value FROM plugin_settings WHERE plugin_slug=?");
  D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:344:            $__s->execute(['effect-santa-hat']);
  D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:345:            while ($__r = $__s->fetch(PDO::FETCH_ASSOC))
{ $__cfg[$__r['setting_key']] = $__r['setting_value']; }
  D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:346:        } catch (Throwable $e) {}
  D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:347:    }
  D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:371:var emoji=<?= json_encode($emoji) ?>;
  D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:372:var imgUrl=<?= json_encode($imgUrl) ?>;
> D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:373:function add(){var l=document.querySelector('.navbar-bran
d,.brand-icon,.logo,header .navbar-brand');if(!l)return;
  D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:374:var w=l.closest('.navbar-brand')||l.parentElement;if(!w)r
eturn;
> D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:375:if(w.querySelector('.we-santa-hat'))return;w.style.positi
on='relative';
  D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:376:var h=document.createElement('div');h.className='we-santa
-hat';
  D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:377:if(useEmoji||!imgUrl){h.textContent=emoji;h.style.fontSiz
e='<?= $size ?>px';h.style.width='auto';h.style.height='auto';}
  D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:378:else{var img=document.createElement('img');img.src=imgUrl
;img.alt='';h.appendChild(img);}
  D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:379:w.appendChild(h);}
  D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:415:    if (isset($pdo) && $pdo instanceof PDO) {
  D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:416:        try {
> D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:417:            $__s = $pdo->prepare("SELECT setting_key,sett
ing_value FROM plugin_settings WHERE plugin_slug=?");
  D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:418:            $__s->execute(['effect-santa-sled']);
  D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:419:            while ($__r = $__s->fetch(PDO::FETCH_ASSOC))
{ $__cfg[$__r['setting_key']] = $__r['setting_value']; }
  D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:420:        } catch (Throwable $e) {}
  D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:421:    }
  D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:475:    if (isset($pdo) && $pdo instanceof PDO) {
  D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:476:        try {
> D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:477:            $__s = $pdo->prepare("SELECT setting_key,sett
ing_value FROM plugin_settings WHERE plugin_slug=?");
  D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:478:            $__s->execute(['effect-christmas-bg']);
  D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:479:            while ($__r = $__s->fetch(PDO::FETCH_ASSOC))
{ $__cfg[$__r['setting_key']] = $__r['setting_value']; }
  D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:480:        } catch (Throwable $e) {}
  D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:481:    }
  D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:539:    if (isset($pdo) && $pdo instanceof PDO) {
  D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:540:        try {
> D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:541:            $__s = $pdo->prepare("SELECT setting_key,sett
ing_value FROM plugin_settings WHERE plugin_slug=?");
  D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:542:            $__s->execute(['effect-christmas-paws']);
  D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:543:            while ($__r = $__s->fetch(PDO::FETCH_ASSOC))
{ $__cfg[$__r['setting_key']] = $__r['setting_value']; }
  D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:544:        } catch (Throwable $e) {}
  D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:545:    }
  D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:557:.we-xmas-paw{position:fixed;pointer-events:none;z-index:9
996;
  D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:558:width:<?= $size ?>px;height:<?= $size ?>px;
> D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:559:opacity:<?= $opacity ?>;transition:opacity 1s ease;user-s
elect:none}
  D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:560:.we-xmas-paw img{width:100%;height:100%;display:block;obj
ect-fit:contain;filter:drop-shadow(0 2px 4px rgba(0,0,0,0.3))}
  D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:561:.we-xmas-paw.fade-out{opacity:0}
  D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:562:</style>
  D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:563:<script>
  D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:615:    if (isset($pdo) && $pdo instanceof PDO) {
  D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:616:        try {
> D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:617:            $__s = $pdo->prepare("SELECT setting_key,sett
ing_value FROM plugin_settings WHERE plugin_slug=?");
  D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:618:            $__s->execute(['effect-christmas-tree']);
  D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:619:            while ($__r = $__s->fetch(PDO::FETCH_ASSOC))
{ $__cfg[$__r['setting_key']] = $__r['setting_value']; }
  D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:620:        } catch (Throwable $e) {}
  D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:621:    }
  D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:634:z-index:9995;pointer-events:none;
  D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:635:filter:drop-shadow(0 6px 12px rgba(0,0,0,0.5));
> D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:636:user-select:none;<?= $sway ? 'animation:weTreeSway 3s eas
e-in-out infinite alternate;' : '' ?>}
  D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:637:.we-xmas-tree img{width:<?= $size ?>px;height:auto;displa
y:block}
  D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:638:.we-xmas-tree-emoji{font-size:<?= $size ?>px;display:bloc
k}
  D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:639:@keyframes weTreeSway{from{transform:rotate(-3deg)}to{tra
nsform:rotate(3deg)}}
  D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:640:</style>
  D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:677:    if (isset($pdo) && $pdo instanceof PDO) {
  D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:678:        try {
> D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:679:            $__s = $pdo->prepare("SELECT setting_key,sett
ing_value FROM plugin_settings WHERE plugin_slug=?");
  D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:680:            $__s->execute(['effect-train']);
  D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:681:            while ($__r = $__s->fetch(PDO::FETCH_ASSOC))
{ $__cfg[$__r['setting_key']] = $__r['setting_value']; }
  D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:682:        } catch (Throwable $e) {}
  D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:683:    }
  D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:736:    if (isset($pdo) && $pdo instanceof PDO) {
  D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:737:        try {
> D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:738:            $__s = $pdo->prepare("SELECT setting_key,sett
ing_value FROM plugin_settings WHERE plugin_slug=?");
  D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:739:            $__s->execute(['effect-lightning']);
  D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:740:            while ($__r = $__s->fetch(PDO::FETCH_ASSOC))
{ $__cfg[$__r['setting_key']] = $__r['setting_value']; }
  D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:741:        } catch (Throwable $e) {}
  D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:742:    }
  D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:758:(function(){var f=document.getElementById('weLightningFla
sh');if(!f)return;
  D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:759:var interval=<?= $interval * 1000 ?>;
> D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:760:function strike(){f.classList.remove('active');void f.off
setWidth;f.classList.add('active');
  D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:761:setTimeout(strike,interval+Math.random()*interval);}
  D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:762:setTimeout(strike,Math.random()*interval);})();
  D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:763:</script>
  D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:764:        <?php
  D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:795:    if (isset($pdo) && $pdo instanceof PDO) {
  D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:796:        try {
> D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:797:            $__s = $pdo->prepare("SELECT setting_key,sett
ing_value FROM plugin_settings WHERE plugin_slug=?");
  D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:798:            $__s->execute(['effect-leaves']);
  D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:799:            while ($__r = $__s->fetch(PDO::FETCH_ASSOC))
{ $__cfg[$__r['setting_key']] = $__r['setting_value']; }
  D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:800:        } catch (Throwable $e) {}
  D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:801:    }
  D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:870:    if (isset($pdo) && $pdo instanceof PDO) {
  D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:871:        try {
> D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:872:            $__s = $pdo->prepare("SELECT setting_key,sett
ing_value FROM plugin_settings WHERE plugin_slug=?");
  D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:873:            $__s->execute(['effect-gifts']);
  D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:874:            while ($__r = $__s->fetch(PDO::FETCH_ASSOC))
{ $__cfg[$__r['setting_key']] = $__r['setting_value']; }
  D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:875:        } catch (Throwable $e) {}
  D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:876:    }
  D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:946:    if (isset($pdo) && $pdo instanceof PDO) {
  D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:947:        try {
> D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:948:            $__s = $pdo->prepare("SELECT setting_key,sett
ing_value FROM plugin_settings WHERE plugin_slug=?");
  D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:949:            $__s->execute(['effect-witch']);
  D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:950:            while ($__r = $__s->fetch(PDO::FETCH_ASSOC))
{ $__cfg[$__r['setting_key']] = $__r['setting_value']; }
  D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:951:        } catch (Throwable $e) {}
  D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:952:    }
  D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:1010:    if (isset($pdo) && $pdo instanceof PDO) {
  D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:1011:        try {
> D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:1012:            $__s = $pdo->prepare("SELECT setting_key,set
ting_value FROM plugin_settings WHERE plugin_slug=?");
  D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:1013:            $__s->execute(['effect-c4-bomb']);
  D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:1014:            while ($__r = $__s->fetch(PDO::FETCH_ASSOC))
 { $__cfg[$__r['setting_key']] = $__r['setting_value']; }
  D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:1015:        } catch (Throwable $e) {}
  D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:1016:    }
  D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:1063:   =====================================================
===== */
  D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:1064:
> D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:1065:function generateAllEffects(PDO $pdo, string $pluginsDir
): array {
  D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:1066:    $effects = getEffectPlugins();
  D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:1067:    $results = ['created' => 0, 'skipped' => 0, 'errors'
 => [], 'files_written' => 0];
  D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:1068:
  D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:1069:    foreach ($effects as $slug => $data) {
  D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:1092:
  D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:1093:        if (@file_put_contents(
> D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:1094:            $pluginDir . '/plugin.json',
  D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:1095:            json_encode($manifest, JSON_PRETTY_PRINT | J
SON_UNESCAPED_UNICODE)
  D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:1096:        ) !== false) {
  D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:1097:            $results['files_written']++;
  D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:1098:        }
  D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:1102:        }
  D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:1103:
> D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:1104:        $result = registerPlugin($pdo, $meta, $settings)
;
  D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:1105:
  D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:1106:        if ($result['status'] === 'created') $results['c
reated']++;
  D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:1107:        elseif ($result['status'] === 'skip') $results['
skipped']++;
  D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:1108:        else $results['errors'][] = $result['error'] ??
'Unknown error';
  D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:1126:
  D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:1127:    try {
> D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:1128:        $stmt = $pdo->prepare("SELECT * FROM plugins WHE
RE id = ? LIMIT 1");
  D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:1129:        $stmt->execute([$id]);
  D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:1130:        $p = $stmt->fetch(PDO::FETCH_ASSOC);
  D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:1131:
  D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:1132:        if (!$p) {
  D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:1143:        $activatedAt = $newStatus === 'active' ? date('Y
-m-d H:i:s') : null;
  D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:1144:
> D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:1145:        $pdo->prepare("UPDATE plugins SET status = ?, ac
tivated_at = ? WHERE id = ?")
  D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:1146:            ->execute([$newStatus, $activatedAt, $id]);
  D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:1147:
  D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:1148:        echo json_encode([
  D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:1149:            'ok'      => true,
  D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:1168:
  D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:1169:    if ($action === 'generate_all_effects') {
> D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:1170:        $res = generateAllEffects($pdo, $pluginsDir);
  D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:1171:        $msg = "Created: {$res['created']} - Skipped: {$
res['skipped']} - Files: {$res['files_written']}";
  D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:1172:        if (!empty($res['errors'])) $msg .= ' | Errors:
' . implode(', ', array_slice($res['errors'], 0, 3));
  D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:1173:        $flash = ['type' => $res['created'] > 0 ? 'succe
ss' : 'info', 'message' => $msg];
  D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:1174:    }
  D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:1175:
> D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:1176:    elseif ($action === 'delete') {
  D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:1177:        $id = (int)($_POST['id'] ?? 0);
  D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:1178:        $removeFiles = isset($_POST['remove_files']) &&
$_POST['remove_files'] === '1';
  D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:1179:
  D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:1180:        if ($id > 0) {
> D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:1181:            $stmt = $pdo->prepare("SELECT * FROM plugins
 WHERE id = ? LIMIT 1");
  D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:1182:            $stmt->execute([$id]);
  D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:1183:            $p = $stmt->fetch(PDO::FETCH_ASSOC);
  D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:1184:
  D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:1185:            if ($p) {
  D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:1186:                if ((int)$p['is_required'] === 1) {
> D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:1187:                    $flash = ['type' => 'error', 'messag
e' => 'Cannot delete required plugin.'];
  D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:1188:                } else {
> D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:1189:                    $pdo->prepare("DELETE FROM plugin_se
ttings WHERE plugin_slug = ?")->execute([$p['slug']]);
> D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:1190:                    $pdo->prepare("DELETE FROM plugins W
HERE id = ?")->execute([$id]);
  D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:1191:
  D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:1192:                    if ($removeFiles) {
  D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:1193:                        $path = $pluginsDir . '/' . $p['
slug'];
  D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:1194:                        if (is_dir($path)) {
  D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:1204:                        }
  D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:1205:                    }
> D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:1206:                    $flash = ['type' => 'success', 'mess
age' => "Plugin '{$p['name']}' deleted."];
  D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:1207:                }
  D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:1208:            }
  D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:1209:        }
  D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:1210:    }
  D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:1211:
> D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:1212:    elseif ($action === 'delete_all_effects') {
  D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:1213:        $effects = getEffectPlugins();
> D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:1214:        $deleted = 0;
  D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:1215:        foreach (array_keys($effects) as $slug) {
  D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:1216:            try {
> D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:1217:                $pdo->prepare("DELETE FROM plugin_settin
gs WHERE plugin_slug = ?")->execute([$slug]);
> D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:1218:                $stmt = $pdo->prepare("DELETE FROM plugi
ns WHERE slug = ?");
  D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:1219:                $stmt->execute([$slug]);
> D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:1220:                $deleted += $stmt->rowCount();
  D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:1221:
  D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:1222:                $path = $pluginsDir . '/' . $slug;
  D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:1223:                if (is_dir($path)) {
  D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:1224:                    $it = new RecursiveIteratorIterator(
  D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:1234:            } catch (Throwable $e) {}
  D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:1235:        }
> D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:1236:        $flash = ['type' => 'success', 'message' => "Del
eted $deleted effect plugin(s)."];
  D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:1237:    }
  D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:1238:
  D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:1239:    elseif ($action === 'save_settings') {
  D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:1240:        $slug = trim((string)($_POST['slug'] ?? ''));
  D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:1247:                try {
  D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:1248:                    $pdo->prepare("
> D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:1249:                        INSERT INTO plugin_settings (plu
gin_slug, setting_key, setting_value)
  D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:1250:                        VALUES (?, ?, ?)
> D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:1251:                        ON DUPLICATE KEY UPDATE setting_
value = VALUES(setting_value)
  D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:1252:                    ")->execute([$slug, $k, (string)$v])
;
  D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:1253:                } catch (Throwable $e) {}
  D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:1254:            }
  D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:1255:            $flash = ['type' => 'success', 'message' =>
'Settings saved.'];
  D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:1295:$plugins = [];
  D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:1296:try {
> D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:1297:    $stmt = $pdo->prepare("SELECT * FROM plugins $whereS
ql ORDER BY status DESC, name ASC");
  D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:1298:    $stmt->execute($params);
  D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:1299:    $plugins = $stmt->fetchAll(PDO::FETCH_ASSOC);
  D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:1300:} catch (Throwable $e) {}
  D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:1301:
  D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:1306:
  D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:1307:try {
> D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:1308:    $statTotal    = (int)$pdo->query("SELECT COUNT(*) FR
OM plugins")->fetchColumn();
> D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:1309:    $statActive   = (int)$pdo->query("SELECT COUNT(*) FR
OM plugins WHERE status = 'active'")->fetchColumn();
> D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:1310:    $statInactive = (int)$pdo->query("SELECT COUNT(*) FR
OM plugins WHERE status = 'inactive'")->fetchColumn();
> D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:1311:    $statEffects  = (int)$pdo->query("SELECT COUNT(*) FR
OM plugins WHERE slug LIKE 'effect-%'")->fetchColumn();
  D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:1312:} catch (Throwable $e) {}
  D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:1313:
  D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:1314:$viewPlugin = null;
  D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:1315:$viewSettings = [];
  D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:1318:    $id = (int)$_GET['view'];
  D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:1319:    try {
> D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:1320:        $stmt = $pdo->prepare("SELECT * FROM plugins WHE
RE id = ? LIMIT 1");
  D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:1321:        $stmt->execute([$id]);
  D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:1322:        $viewPlugin = $stmt->fetch(PDO::FETCH_ASSOC);
  D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:1323:
  D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:1324:        if ($viewPlugin) {
> D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:1325:            $stmt = $pdo->prepare("SELECT * FROM plugin_
settings WHERE plugin_slug = ? ORDER BY setting_key ASC");
  D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:1326:            $stmt->execute([$viewPlugin['slug']]);
  D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:1327:            $viewSettings = $stmt->fetchAll(PDO::FETCH_A
SSOC);
  D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:1328:        }
  D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:1329:    } catch (Throwable $e) {}
  D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:1491:        transform: scale(1.03);
  D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:1492:    }
> D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:1493:    .bg-preset.selected {
  D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:1494:        border-color: #10b981;
  D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:1495:        box-shadow: 0 0 0 3px rgba(16, 185, 129, .3);
  D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:1496:    }
  D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:1497:    .bg-preset-label {
  D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:1632:                    <i class="fa-solid fa-puzzle-piece w
-5"></i><span>Plugins</span>
  D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:1633:                </a>
> D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:1634:                <a href="updates.php" class="flex items-
center gap-3 px-4 py-3 rounded-xl text-slate-300 hover:bg-slate-800/60 hover:text-emerald-400 transition">
> D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:1635:                    <i class="fa-solid fa-cloud-arrow-do
wn w-5"></i><span>Update</span>
  D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:1636:                </a>
  D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:1637:                <a href="api.php" class="flex items-cent
er gap-3 px-4 py-3 rounded-xl text-slate-300 hover:bg-slate-800/60 hover:text-cyan-400 transition">
  D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:1638:                    <i class="fa-solid fa-plug w-5"></i>
<span>API</span>
  D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:1639:                </a>
  D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:1732:
  D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:1733:            <script>
> D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:1734:                (function () {
  D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:1735:                    var flash = document.getElementById(
'plgFlash');
  D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:1736:                    if (!flash) return;
  D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:1737:
> D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:1738:                    var autoTimer = setTimeout(function
() {
  D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:1739:                        hidePlgFlash();
  D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:1740:                    }, 2000);
  D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:1741:
> D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:1742:                    window.hidePlgFlash = function () {
  D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:1743:                        clearTimeout(autoTimer);
  D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:1744:                        if (!flash || flash.classList.co
ntains('plg-flash-hide')) return;
  D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:1745:                        flash.classList.add('plg-flash-h
ide');
  D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:1746:
> D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:1747:                        setTimeout(function () {
  D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:1748:                            if (flash && flash.parentNod
e) {
  D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:1749:                                flash.parentNode.removeC
hild(flash);
  D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:1750:                            }
  D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:1751:                        }, 500);
  D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:1781:
  D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:1782:                    <?php if ($statEffects > 0): ?>
> D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:1783:                        <form method="post" class="inlin
e" data-confirm="Delete ALL effect plugins?">
> D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:1784:                            <input type="hidden" name="a
ction" value="delete_all_effects">
  D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:1785:                            <button type="submit"
  D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:1786:                                    class="px-4 py-3 rou
nded-xl bg-slate-800 hover:bg-slate-700 text-rose-300 font-bold text-sm transition">
> D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:1787:                                <i class="fa-solid fa-tr
ash mr-1"></i>Delete All
  D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:1788:                            </button>
  D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:1789:                        </form>
  D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:1790:                    <?php endif; ?>
  D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:1791:                </div>
  D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:1890:                                         style="backgrou
nd-image: url('<?= htmlspecialchars($url, ENT_QUOTES) ?>');"
  D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:1891:                                         data-url="<?= h
tmlspecialchars($url, ENT_QUOTES) ?>"
> D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:1892:                                         onclick="select
BgPreset(this)">
  D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:1893:                                        <div class="bg-p
reset-label"><?= htmlspecialchars($label) ?></div>
  D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:1894:                                    </div>
  D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:1895:                                <?php endforeach; ?>
  D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:1896:                            </div>
  D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:1923:                                                  rows="
2"
  D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:1924:                                                  class=
"w-full px-4 py-3 rounded-xl bg-slate-900/60 border border-slate-700 focus:border-indigo-500 outline-none text-white te
xt-sm font-mono"
> D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:1925:                                                  oninpu
t="updatePreview('<?= htmlspecialchars($key) ?>', this.value)"><?= htmlspecialchars($val) ?></textarea>
  D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:1926:                                    <?php else: ?>
  D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:1927:                                        <input type="tex
t" name="settings[<?= htmlspecialchars($key) ?>]"
  D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:1928:                                               value="<?
= htmlspecialchars($val) ?>"
  D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:1929:                                               class="w-
full px-4 py-3 rounded-xl bg-slate-900/60 border border-slate-700 focus:border-indigo-500 outline-none text-white text-
sm <?= $isImageUrl ? 'font-mono' : '' ?>"
> D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:1930:                                               oninput="
updatePreview('<?= htmlspecialchars($key) ?>', this.value)">
  D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:1931:                                    <?php endif; ?>
  D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:1932:                                </div>
  D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:1933:                            <?php endforeach; ?>
  D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:1934:
  D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:1954:                </div>
  D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:1955:
> D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:1956:                <select name="status" class="px-4 py-3 r
ounded-xl bg-slate-900/60 border border-slate-700 outline-none text-white text-sm">
  D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:1957:                    <option value="">All Statuses</optio
n>
> D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:1958:                    <option value="active"   <?= $filter
Status === 'active' ? 'selected' : '' ?>>Active</option>
> D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:1959:                    <option value="inactive" <?= $filter
Status === 'inactive' ? 'selected' : '' ?>>Inactive</option>
> D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:1960:                </select>
  D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:1961:
  D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:1962:                <button type="submit" class="px-6 py-3 r
ounded-xl bg-gradient-to-r from-indigo-600 to-purple-600 hover:from-indigo-500 hover:to-purple-500 text-white font-bold
 text-sm shadow-lg transition">
  D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:1963:                    <i class="fa-solid fa-filter mr-2"><
/i>Filter
  D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:1964:                </button>
  D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:2035:                                </a>
  D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:2036:
> D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:2037:                                <form method="post" clas
s="inline ml-auto" data-confirm="Delete plugin?">
> D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:2038:                                    <input type="hidden"
 name="action" value="delete">
  D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:2039:                                    <input type="hidden"
 name="id" value="<?= (int)$p['id'] ?>">
  D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:2040:                                    <input type="hidden"
 name="remove_files" value="1">
  D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:2041:                                    <button type="submit
" class="p-2 rounded-lg text-slate-400 hover:text-rose-400 hover:bg-rose-500/10 transition">
  D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:2042:                                        <i class="fa-sol
id fa-trash text-xs"></i>
  D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:2119:    const close   = document.getElementById('close-sideb
ar');
  D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:2120:
> D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:2121:    function toggleSidebar() {
  D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:2122:        sidebar.classList.toggle('-translate-x-full');
  D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:2123:        overlay.classList.toggle('hidden');
  D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:2124:    }
  D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:2125:    open?.addEventListener('click', toggleSidebar);
  D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:2142:        var pendingForm = null;
  D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:2143:
> D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:2144:        function openConfirm(form, message) {
  D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:2145:            pendingForm = form;
  D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:2146:            modalText.textContent = message || 'Do you r
eally want to do this action?';
  D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:2147:            modal.classList.add('active');
  D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:2148:        }
  D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:2149:
> D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:2150:        function closeConfirm() {
  D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:2151:            modal.classList.remove('active');
  D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:2152:            pendingForm = null;
  D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:2153:        }
  D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:2154:
  D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:2175:            if (!form.dataset.confirm) return;
  D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:2176:            if (form.dataset.confirmed === '1') {
> D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:2177:                delete form.dataset.confirmed;
  D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:2178:                return;
  D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:2179:            }
  D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:2180:            e.preventDefault();
  D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:2181:            openConfirm(form, form.dataset.confirm);
  D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:2186:       AJAX TOGGLE (no page reload, no jump to top)
  D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:2187:       =================================================
========= */
> D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:2188:    function ajaxToggle(checkbox, pluginId) {
  D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:2189:        var label   = checkbox.closest('.plg-switch');
> D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:2190:        var card    = document.querySelector('[data-plug
in-card="' + pluginId + '"]');
> D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:2191:        var dot     = card ? card.querySelector('.plg-ac
tive-dot') : null;
  D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:2192:        var prevVal = !checkbox.checked;
  D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:2193:
  D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:2194:        if (label) label.classList.add('loading');
  D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:2195:
  D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:2217:            var isActive = data.status === 'active';
  D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:2218:
> D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:2219:            /* Update active dot */
  D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:2220:            if (dot) {
  D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:2221:                if (isActive) dot.classList.remove('hidd
en');
  D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:2222:                else          dot.classList.add('hidden'
);
  D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:2223:            }
  D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:2224:
> D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:2225:            /* Update card border */
  D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:2226:            if (card) {
  D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:2227:                card.classList.toggle('border-emerald-50
0/40', isActive);
  D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:2228:                card.classList.toggle('border-slate-700/
50',   !isActive);
  D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:2229:            }
  D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:2230:
> D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:2231:            /* Update stats counters */
> D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:2232:            updateStats(isActive);
  D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:2233:
  D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:2234:            /* Toast notification (auto-hide after 2s) *
/
  D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:2235:            showToast(data.message || 'Done', 'success')
;
  D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:2236:        })
  D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:2242:    }
  D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:2243:
> D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:2244:    /* Update active/inactive counters without page relo
ad */
> D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:2245:    function updateStats(isActive) {
  D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:2246:        var actEl  = document.getElementById('statActive
');
  D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:2247:        var inaEl  = document.getElementById('statInacti
ve');
  D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:2248:        if (!actEl || !inaEl) return;
  D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:2249:
  D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:2261:       TOAST (auto-hide after 2s, animated)
  D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:2262:       =================================================
========= */
> D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:2263:    function showToast(message, type) {
  D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:2264:        type = type || 'success';
  D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:2265:        var container = document.getElementById('plgToas
t');
  D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:2266:        if (!container) return;
  D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:2267:
  D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:2276:        toast.innerHTML = '<i class="fa-solid ' + (icons
[type] || icons.success) + '"></i>' +
  D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:2277:                          '<span></span>';
> D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:2278:        toast.querySelector('span').textContent = messag
e;
  D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:2279:
  D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:2280:        container.appendChild(toast);
  D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:2281:
  D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:2282:        /* Slide in */
  D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:2306:
  D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:2307:    /* =================================================
=========
> D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:2308:       BACKGROUND PRESET SELECTION
  D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:2309:       =================================================
========= */
> D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:2310:    function selectBgPreset(el) {
  D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:2311:        var url = el.dataset.url;
> D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:2312:        var input = document.querySelector('input[name="
settings[image_url]"]')
> D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:2313:                || document.querySelector('textarea[name
="settings[image_url]"]');
  D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:2314:
  D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:2315:        if (input) {
  D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:2316:            input.value = url;
  D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:2317:            input.dispatchEvent(new Event('input', { bub
bles: true }));
  D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:2323:        }
  D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:2324:
> D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:2325:        document.querySelectorAll('.bg-preset').forEach(
function(p) {
> D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:2326:            p.classList.remove('selected');
  D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:2327:        });
> D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:2328:        el.classList.add('selected');
  D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:2329:    }
  D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:2330:
  D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:2331:    /* =================================================
=========
  D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:2332:       LIVE PREVIEW FOR BACKGROUND
  D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:2333:       =================================================
========= */
> D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:2334:    function updatePreview(key, value) {
  D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:2335:        if (key !== 'image_url') return;
  D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:2336:        var preview = document.getElementById('bgPreview
');
  D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:2337:        if (!preview) return;
  D:\Program Files\XAMPP\htdocs\webengine\adm\plugins.php:2338:        if (value && value.trim() !== '') {

===== PLUGIN ENTRY POINTS =====

--- effect-c4-bomb ---
<?php
declare(strict_types=1);
if (!defined('WE_EFFECT_C4_LOADED')) {
    define('WE_EFFECT_C4_LOADED', true);
    $__cfg = ['enabled'=>'1','image_url'=>'https://cdn-icons-png.flaticon.com/512/3079/3079508.png','use_emoji'=>'0','emoji'=>'ðŸ’£','size'=>'100','position'=>'br','blink'=>'1','glow_color'=>'#ff0000'];
    if (isset($pdo) && $pdo instanceof PDO) {
        try {
            $__s = $pdo->prepare("SELECT setting_key,setting_value FROM plugin_settings WHERE plugin_slug=?");
            $__s->execute(['effect-c4-bomb']);
            while ($__r = $__s->fetch(PDO::FETCH_ASSOC)) { $__cfg[$__r['setting_key']] = $__r['setting_value']; }
        } catch (Throwable $e) {}
    }
    if (($__cfg['enabled'] ?? '1') !== '1') return;
    add_action('footer', function() use ($__cfg) {
        $imgUrl  = htmlspecialchars($__cfg['image_url'], ENT_QUOTES);
        $useEmoji= ($__cfg['use_emoji'] ?? '0') === '1';
        $emoji   = $__cfg['emoji'] ?: 'ðŸ’£';
        $size    = max(40, min(300, (int)$__cfg['size']));
        $pos     = $__cfg['position'] ?: 'br';
        $blink   = ($__cfg['blink'] ?? '1') === '1';
        $glow    = htmlspecialchars($__cfg['glow_color'], ENT_QUOTES);

        $posCSS = '';
        switch ($pos) {
            case 'br': $posCSS = 'bottom:25px;right:25px'; break;
            case 'bl': $posCSS = 'bottom:25px;left:25px'; break;
            case 'tr': $posCSS = 'top:25px;right:25px'; break;
            case 'tl': $posCSS = 'top:25px;left:25px'; break;
            case 'center': $posCSS = 'top:50%;left:50%;transform:translate(-50%,-50%)'; break;
            default: $posCSS = 'bottom:25px;right:25px';
        }
        ?>
<style>
.we-c4{position:fixed;<?= $posCSS ?>;z-index:9995;pointer-events:none;
filter:drop-shadow(0 0 20px <?= $glow ?>);
<?= $blink ? 'animation:weC4Blink 0.85s infinite alternate;' : '' ?>}
.we-c4 img{width:<?= $size ?>px;height:auto;display:block}
.we-c4-emoji{font-size:<?= $size ?>px;display:block}
@keyframes weC4Blink{
0%{filter:drop-shadow(0 0 6px <?= $glow ?>);transform:scale(1)}
100%{filter:drop-shadow(0 0 35px <?= $glow ?>);transform:scale(1.08)}}
</style>
<?php if ($useEmoji || $imgUrl === ''): ?>
<div class="we-c4"><span class="we-c4-emoji"><?= htmlspecialchars($emoji) ?></span></div>
<?php else: ?>

--- effect-christmas-bg ---
<?php
declare(strict_types=1);
if (!defined('WE_EFFECT_XMAS_BG_LOADED')) {
    define('WE_EFFECT_XMAS_BG_LOADED', true);
    $__cfg = ['enabled'=>'1','image_url'=>'','opacity'=>'0.5','blur'=>'0','fixed'=>'1','darken'=>'0.35'];
    if (isset($pdo) && $pdo instanceof PDO) {
        try {
            $__s = $pdo->prepare("SELECT setting_key,setting_value FROM plugin_settings WHERE plugin_slug=?");
            $__s->execute(['effect-christmas-bg']);
            while ($__r = $__s->fetch(PDO::FETCH_ASSOC)) { $__cfg[$__r['setting_key']] = $__r['setting_value']; }
        } catch (Throwable $e) {}
    }
    if (($__cfg['enabled'] ?? '1') !== '1') return;
    add_action('head', function() use ($__cfg) {
        $img    = trim((string)$__cfg['image_url']);
        $op     = max(0, min(1, (float)$__cfg['opacity']));
        $blur   = max(0, min(20, (int)$__cfg['blur']));
        $fixed  = ($__cfg['fixed'] ?? '1') === '1' ? 'fixed' : 'scroll';
        $darken = max(0, min(1, (float)$__cfg['darken']));
        if ($img === '') return;
        $imgSafe = htmlspecialchars($img, ENT_QUOTES);
        ?>
<style>
html, body {
    background-image: url('<?= $imgSafe ?>') !important;
    background-size: cover !important;
    background-position: center center !important;
    background-repeat: no-repeat !important;
    background-attachment: <?= $fixed ?> !important;
}
body::before {
    background: rgba(0, 0, 0, <?= $darken ?>) !important;
    backdrop-filter: <?= $blur > 0 ? 'blur(' . $blur . 'px)' : 'none' ?> !important;
    -webkit-backdrop-filter: <?= $blur > 0 ? 'blur(' . $blur . 'px)' : 'none' ?> !important;
}
.bg-layer, .bg-overlay { opacity: <?= $op ?>; }
</style>
        <?php
    });
}

--- effect-christmas-paws ---
<?php
declare(strict_types=1);
if (!defined('WE_EFFECT_XMAS_PAWS_LOADED')) {
    define('WE_EFFECT_XMAS_PAWS_LOADED', true);
    $__cfg = ['enabled'=>'1','image_url'=>'https://cdn-icons-png.flaticon.com/512/1998/1998592.png','use_emoji'=>'0','emoji'=>'ðŸ¾','size'=>'32','opacity'=>'0.65','lifetime'=>'2','min_dist'=>'70'];
    if (isset($pdo) && $pdo instanceof PDO) {
        try {
            $__s = $pdo->prepare("SELECT setting_key,setting_value FROM plugin_settings WHERE plugin_slug=?");
            $__s->execute(['effect-christmas-paws']);
            while ($__r = $__s->fetch(PDO::FETCH_ASSOC)) { $__cfg[$__r['setting_key']] = $__r['setting_value']; }
        } catch (Throwable $e) {}
    }
    if (($__cfg['enabled'] ?? '1') !== '1') return;
    add_action('footer', function() use ($__cfg) {
        $imgUrl  = htmlspecialchars($__cfg['image_url'], ENT_QUOTES);
        $useEmoji= ($__cfg['use_emoji'] ?? '0') === '1';
        $emoji   = $__cfg['emoji'] ?: 'ðŸ¾';
        $size    = max(16, min(120, (int)$__cfg['size']));
        $opacity = max(0.1, min(1, (float)$__cfg['opacity']));
        $life    = max(0.5, min(15, (float)$__cfg['lifetime'])) * 1000;
        $dist    = max(20, min(300, (int)$__cfg['min_dist']));
        ?>
<style>
.we-xmas-paw{position:fixed;pointer-events:none;z-index:9996;
width:<?= $size ?>px;height:<?= $size ?>px;
opacity:<?= $opacity ?>;transition:opacity 1s ease;user-select:none}
.we-xmas-paw img{width:100%;height:100%;display:block;object-fit:contain;filter:drop-shadow(0 2px 4px rgba(0,0,0,0.3))}
.we-xmas-paw.fade-out{opacity:0}
</style>
<script>
(function(){
var useEmoji=<?= $useEmoji ? 'true' : 'false' ?>;
var emoji=<?= json_encode($emoji) ?>;
var imgUrl=<?= json_encode($imgUrl) ?>;
var life=<?= $life ?>;
var minD=<?= $dist ?>;
var lastX=-999,lastY=-999;
document.addEventListener('mousemove',function(e){
var d=Math.hypot(e.clientX-lastX,e.clientY-lastY);
if(d<minD)return;
lastX=e.clientX;lastY=e.clientY;
var p=document.createElement('div');p.className='we-xmas-paw';
p.style.left=(e.clientX-<?= $size / 2 ?>)+'px';p.style.top=(e.clientY-<?= $size / 2 ?>)+'px';
if(useEmoji||!imgUrl){p.textContent=emoji;p.style.fontSize='<?= $size ?>px';p.style.width='auto';p.style.height='auto';}
else{var img=document.createElement('img');img.src=imgUrl;img.alt='';p.appendChild(img);}

--- effect-christmas-tree ---
<?php
declare(strict_types=1);
if (!defined('WE_EFFECT_XMAS_TREE_LOADED')) {
    define('WE_EFFECT_XMAS_TREE_LOADED', true);
    $__cfg = ['enabled'=>'1','image_url'=>'https://cdn-icons-png.flaticon.com/512/2941/2941733.png','use_emoji'=>'0','emoji'=>'ðŸŽ„','size'=>'140','bottom'=>'20','left'=>'20','sway'=>'1'];
    if (isset($pdo) && $pdo instanceof PDO) {
        try {
            $__s = $pdo->prepare("SELECT setting_key,setting_value FROM plugin_settings WHERE plugin_slug=?");
            $__s->execute(['effect-christmas-tree']);
            while ($__r = $__s->fetch(PDO::FETCH_ASSOC)) { $__cfg[$__r['setting_key']] = $__r['setting_value']; }
        } catch (Throwable $e) {}
    }
    if (($__cfg['enabled'] ?? '1') !== '1') return;
    add_action('footer', function() use ($__cfg) {
        $imgUrl   = htmlspecialchars($__cfg['image_url'], ENT_QUOTES);
        $useEmoji = ($__cfg['use_emoji'] ?? '0') === '1';
        $emoji    = $__cfg['emoji'] ?: 'ðŸŽ„';
        $size     = max(40, min(400, (int)$__cfg['size']));
        $bottom   = max(0, min(500, (int)$__cfg['bottom']));
        $left     = max(0, min(500, (int)$__cfg['left']));
        $sway     = ($__cfg['sway'] ?? '1') === '1';
        ?>
<style>
.we-xmas-tree{position:fixed;bottom:<?= $bottom ?>px;left:<?= $left ?>px;
z-index:9995;pointer-events:none;
filter:drop-shadow(0 6px 12px rgba(0,0,0,0.5));
user-select:none;<?= $sway ? 'animation:weTreeSway 3s ease-in-out infinite alternate;' : '' ?>}
.we-xmas-tree img{width:<?= $size ?>px;height:auto;display:block}
.we-xmas-tree-emoji{font-size:<?= $size ?>px;display:block}
@keyframes weTreeSway{from{transform:rotate(-3deg)}to{transform:rotate(3deg)}}
</style>
<?php if ($useEmoji || $imgUrl === ''): ?>
<div class="we-xmas-tree"><span class="we-xmas-tree-emoji"><?= htmlspecialchars($emoji) ?></span></div>
<?php else: ?>
<div class="we-xmas-tree"><img src="<?= $imgUrl ?>" alt="" loading="lazy"></div>
<?php endif; ?>
        <?php
    });
}

--- effect-gifts ---
<?php
declare(strict_types=1);
if (!defined('WE_EFFECT_GIFTS_LOADED')) {
    define('WE_EFFECT_GIFTS_LOADED', true);
    $__cfg = ['enabled'=>'1','intensity'=>'25','speed'=>'8','size'=>'70','image_urls'=>'','use_emoji'=>'0','emojis'=>'ðŸŽ,ðŸŽ€,ðŸ§¸'];
    if (isset($pdo) && $pdo instanceof PDO) {
        try {
            $__s = $pdo->prepare("SELECT setting_key,setting_value FROM plugin_settings WHERE plugin_slug=?");
            $__s->execute(['effect-gifts']);
            while ($__r = $__s->fetch(PDO::FETCH_ASSOC)) { $__cfg[$__r['setting_key']] = $__r['setting_value']; }
        } catch (Throwable $e) {}
    }
    if (($__cfg['enabled'] ?? '1') !== '1') return;
    add_action('footer', function() use ($__cfg) {
        $intensity = max(1, min(200, (int)$__cfg['intensity']));
        $speed     = max(3, min(30, (float)$__cfg['speed']));
        $size      = max(20, min(200, (int)$__cfg['size']));
        $useEmoji  = ($__cfg['use_emoji'] ?? '0') === '1';
        $urls      = array_values(array_filter(array_map('trim', explode(',', $__cfg['image_urls']))));
        $emojis    = array_values(array_filter(array_map('trim', explode(',', $__cfg['emojis']))));
        if (empty($emojis)) $emojis = ['ðŸŽ'];
        $useUrls = !$useEmoji && !empty($urls);
        ?>
<style>
.we-gift-container{position:fixed;inset:0;pointer-events:none;z-index:9998;overflow:hidden}
.we-gift{position:absolute;top:-15%;width:<?= $size ?>px;height:<?= $size ?>px;
animation:weGiftFall linear infinite;filter:drop-shadow(0 4px 8px rgba(0,0,0,0.4))}
.we-gift img{width:100%;height:100%;display:block;object-fit:contain}
.we-gift.emoji-mode{font-size:<?= $size ?>px;width:auto;height:auto;line-height:1}
@keyframes weGiftFall{0%{transform:translateY(0) rotate(-10deg)}
50%{transform:translateY(50vh) rotate(10deg)}
100%{transform:translateY(110vh) rotate(-10deg)}}
</style>
<div class="we-gift-container" id="weGiftContainer"></div>
<script>
(function(){var c=document.getElementById('weGiftContainer');if(!c)return;
var useUrls=<?= $useUrls ? 'true' : 'false' ?>;
var urls=<?= json_encode($urls) ?>;
var emojis=<?= json_encode($emojis) ?>;
for(var i=0;i<<?= $intensity ?>;i++){var g=document.createElement('div');g.className='we-gift';
if(useUrls){var img=document.createElement('img');img.src=urls[i%urls.length];img.alt='';img.loading='lazy';g.appendChild(img);}
else{g.classList.add('emoji-mode');g.textContent=emojis[i%emojis.length];}
g.style.left=(Math.random()*100)+'%';
var s=(<?= $size ?>*(0.6+Math.random()*0.7));
g.style.width=s+'px';g.style.height=s+'px';

--- effect-leaves ---
<?php
declare(strict_types=1);
if (!defined('WE_EFFECT_LEAVES_LOADED')) {
    define('WE_EFFECT_LEAVES_LOADED', true);
    $__cfg = ['enabled'=>'1','intensity'=>'40','speed'=>'8','size'=>'50','image_urls'=>'','use_emoji'=>'0','emojis'=>'ðŸ‚,ðŸ,ðŸƒ'];
    if (isset($pdo) && $pdo instanceof PDO) {
        try {
            $__s = $pdo->prepare("SELECT setting_key,setting_value FROM plugin_settings WHERE plugin_slug=?");
            $__s->execute(['effect-leaves']);
            while ($__r = $__s->fetch(PDO::FETCH_ASSOC)) { $__cfg[$__r['setting_key']] = $__r['setting_value']; }
        } catch (Throwable $e) {}
    }
    if (($__cfg['enabled'] ?? '1') !== '1') return;
    add_action('footer', function() use ($__cfg) {
        $intensity = max(1, min(300, (int)$__cfg['intensity']));
        $speed     = max(2, min(30, (float)$__cfg['speed']));
        $size      = max(15, min(150, (int)$__cfg['size']));
        $useEmoji  = ($__cfg['use_emoji'] ?? '0') === '1';
        $urls      = array_values(array_filter(array_map('trim', explode(',', $__cfg['image_urls']))));
        $emojis    = array_values(array_filter(array_map('trim', explode(',', $__cfg['emojis']))));
        if (empty($urls)) $urls = [];
        if (empty($emojis)) $emojis = ['ðŸ‚'];
        $useUrls = !$useEmoji && !empty($urls);
        ?>
<style>
.we-leaf-container{position:fixed;inset:0;pointer-events:none;z-index:9998;overflow:hidden}
.we-leaf{position:absolute;top:-10%;width:<?= $size ?>px;height:<?= $size ?>px;
animation:weLeafFall linear infinite;filter:drop-shadow(0 3px 5px rgba(0,0,0,0.35))}
.we-leaf img{width:100%;height:100%;display:block;object-fit:contain}
.we-leaf.emoji-mode{font-size:<?= $size ?>px;width:auto;height:auto;line-height:1}
@keyframes weLeafFall{0%{transform:translateY(0) rotate(0deg);opacity:1}
100%{transform:translateY(110vh) rotate(720deg);opacity:0.3}}
</style>
<div class="we-leaf-container" id="weLeafContainer"></div>
<script>
(function(){var c=document.getElementById('weLeafContainer');if(!c)return;
var useUrls=<?= $useUrls ? 'true' : 'false' ?>;
var urls=<?= json_encode($urls) ?>;
var emojis=<?= json_encode($emojis) ?>;
for(var i=0;i<<?= $intensity ?>;i++){var l=document.createElement('div');l.className='we-leaf';
if(useUrls){var img=document.createElement('img');img.src=urls[i%urls.length];img.alt='';img.loading='lazy';l.appendChild(img);}
else{l.classList.add('emoji-mode');l.textContent=emojis[i%emojis.length];}
l.style.left=(Math.random()*100)+'%';
var s=(<?= $size ?>*(0.6+Math.random()*0.7));
l.style.width=s+'px';l.style.height=s+'px';

--- effect-lightning ---
<?php
declare(strict_types=1);
if (!defined('WE_EFFECT_LIGHTNING_LOADED')) {
    define('WE_EFFECT_LIGHTNING_LOADED', true);
    $__cfg = ['enabled'=>'1','interval'=>'7','duration'=>'0.4','color'=>'#ffffff','opacity'=>'0.85'];
    if (isset($pdo) && $pdo instanceof PDO) {
        try {
            $__s = $pdo->prepare("SELECT setting_key,setting_value FROM plugin_settings WHERE plugin_slug=?");
            $__s->execute(['effect-lightning']);
            while ($__r = $__s->fetch(PDO::FETCH_ASSOC)) { $__cfg[$__r['setting_key']] = $__r['setting_value']; }
        } catch (Throwable $e) {}
    }
    if (($__cfg['enabled'] ?? '1') !== '1') return;
    add_action('footer', function() use ($__cfg) {
        $interval = max(2, min(60, (float)$__cfg['interval']));
        $duration = max(0.1, min(2, (float)$__cfg['duration']));
        $color    = htmlspecialchars($__cfg['color'], ENT_QUOTES);
        $opacity  = max(0.1, min(1, (float)$__cfg['opacity']));
        ?>
<style>
.we-lightning-flash{position:fixed;inset:0;background:<?= $color ?>;opacity:0;
pointer-events:none;z-index:10000;mix-blend-mode:screen}
.we-lightning-flash.active{animation:weLightning <?= $duration ?>s ease-out}
@keyframes weLightning{0%{opacity:0}8%{opacity:<?= $opacity ?>}15%{opacity:0.15}22%{opacity:<?= $opacity * 0.85 ?>}35%{opacity:0.25}100%{opacity:0}}
</style>
<div class="we-lightning-flash" id="weLightningFlash"></div>
<script>
(function(){var f=document.getElementById('weLightningFlash');if(!f)return;
var interval=<?= $interval * 1000 ?>;
function strike(){f.classList.remove('active');void f.offsetWidth;f.classList.add('active');
setTimeout(strike,interval+Math.random()*interval);}
setTimeout(strike,Math.random()*interval);})();
</script>
        <?php
    });
}

--- effect-rain ---
<?php
declare(strict_types=1);
if (!defined('WE_EFFECT_RAIN_LOADED')) {
    define('WE_EFFECT_RAIN_LOADED', true);
    $__cfg = ['enabled'=>'1','intensity'=>'120','speed'=>'2','color'=>'#93c5fd','opacity'=>'0.65','drop_width'=>'2','drop_height'=>'90'];
    if (isset($pdo) && $pdo instanceof PDO) {
        try {
            $__s = $pdo->prepare("SELECT setting_key,setting_value FROM plugin_settings WHERE plugin_slug=?");
            $__s->execute(['effect-rain']);
            while ($__r = $__s->fetch(PDO::FETCH_ASSOC)) { $__cfg[$__r['setting_key']] = $__r['setting_value']; }
        } catch (Throwable $e) {}
    }
    if (($__cfg['enabled'] ?? '1') !== '1') return;
    add_action('footer', function() use ($__cfg) {
        $intensity = max(1, min(500, (int)$__cfg['intensity']));
        $speed     = max(0.5, min(10, (float)$__cfg['speed']));
        $color     = htmlspecialchars($__cfg['color'], ENT_QUOTES);
        $opacity   = max(0.1, min(1, (float)$__cfg['opacity']));
        $width     = max(1, min(10, (int)$__cfg['drop_width']));
        $height    = max(20, min(300, (int)$__cfg['drop_height']));
        ?>
<style>
.we-rain-container{position:fixed;inset:0;pointer-events:none;z-index:9998;overflow:hidden}
.we-rain-drop{position:absolute;top:-10%;width:<?= $width ?>px;height:<?= $height ?>px;
background:linear-gradient(to bottom,transparent,<?= $color ?>);opacity:<?= $opacity ?>;
border-radius:50% 50% 20% 20%;animation:weRainFall linear infinite}
@keyframes weRainFall{to{transform:translateY(110vh)}}
</style>
<div class="we-rain-container" id="weRainContainer"></div>
<script>
(function(){var c=document.getElementById('weRainContainer');if(!c)return;
for(var i=0;i<<?= $intensity ?>;i++){var d=document.createElement('div');d.className='we-rain-drop';
d.style.left=(Math.random()*100)+'%';d.style.animationDelay=(Math.random()*<?= $speed ?>)+'s';
d.style.animationDuration=(<?= $speed ?>*(0.5+Math.random()))+'s';c.appendChild(d);}})();
</script>
        <?php
    });
}

--- effect-santa-hat ---
<?php
declare(strict_types=1);
if (!defined('WE_EFFECT_SANTA_HAT_LOADED')) {
    define('WE_EFFECT_SANTA_HAT_LOADED', true);
    $__cfg = ['enabled'=>'1','image_url'=>'https://cdn-icons-png.flaticon.com/512/6145/6145967.png','use_emoji'=>'0','emoji'=>'ðŸŽ…','size'=>'80','rotate'=>'-30','top'=>'-30','left'=>'20'];
    if (isset($pdo) && $pdo instanceof PDO) {
        try {
            $__s = $pdo->prepare("SELECT setting_key,setting_value FROM plugin_settings WHERE plugin_slug=?");
            $__s->execute(['effect-santa-hat']);
            while ($__r = $__s->fetch(PDO::FETCH_ASSOC)) { $__cfg[$__r['setting_key']] = $__r['setting_value']; }
        } catch (Throwable $e) {}
    }
    if (($__cfg['enabled'] ?? '1') !== '1') return;
    add_action('footer', function() use ($__cfg) {
        $imgUrl   = htmlspecialchars($__cfg['image_url'], ENT_QUOTES);
        $useEmoji = ($__cfg['use_emoji'] ?? '0') === '1';
        $emoji    = $__cfg['emoji'] ?: 'ðŸŽ…';
        $size     = max(30, min(300, (int)$__cfg['size']));
        $rotate   = (int)$__cfg['rotate'];
        $top      = (int)$__cfg['top'];
        $left     = (int)$__cfg['left'];
        ?>
<style>
.we-santa-hat{position:absolute!important;z-index:9999;pointer-events:none;
width:<?= $size ?>px;height:<?= $size ?>px;
top:<?= $top ?>px;left:<?= $left ?>px;
transform:rotate(<?= $rotate ?>deg);transform-origin:bottom left;
animation:weSantaBounce 2s ease-in-out infinite alternate;
filter:drop-shadow(0 4px 8px rgba(0,0,0,0.35))}
.we-santa-hat img{width:100%;height:100%;display:block;object-fit:contain}
@keyframes weSantaBounce{from{transform:rotate(<?= $rotate ?>deg)}to{transform:rotate(<?= $rotate + 6 ?>deg)}}
</style>
<script>
(function(){
var useEmoji=<?= $useEmoji ? 'true' : 'false' ?>;
var emoji=<?= json_encode($emoji) ?>;
var imgUrl=<?= json_encode($imgUrl) ?>;
function add(){var l=document.querySelector('.navbar-brand,.brand-icon,.logo,header .navbar-brand');if(!l)return;
var w=l.closest('.navbar-brand')||l.parentElement;if(!w)return;
if(w.querySelector('.we-santa-hat'))return;w.style.position='relative';
var h=document.createElement('div');h.className='we-santa-hat';
if(useEmoji||!imgUrl){h.textContent=emoji;h.style.fontSize='<?= $size ?>px';h.style.width='auto';h.style.height='auto';}
else{var img=document.createElement('img');img.src=imgUrl;img.alt='';h.appendChild(img);}
w.appendChild(h);}
if(document.readyState==='loading')document.addEventListener('DOMContentLoaded',add);else add();

--- effect-santa-sled ---
<?php
declare(strict_types=1);
if (!defined('WE_EFFECT_SANTA_SLED_LOADED')) {
    define('WE_EFFECT_SANTA_SLED_LOADED', true);
    $__cfg = ['enabled'=>'1','image_url'=>'https://cdn-icons-png.flaticon.com/512/3467/3467891.png','use_emoji'=>'0','emoji'=>'ðŸ›·ðŸŽ…ðŸŽ','size'=>'160','speed'=>'22','top'=>'15','flip'=>'0'];
    if (isset($pdo) && $pdo instanceof PDO) {
        try {
            $__s = $pdo->prepare("SELECT setting_key,setting_value FROM plugin_settings WHERE plugin_slug=?");
            $__s->execute(['effect-santa-sled']);
            while ($__r = $__s->fetch(PDO::FETCH_ASSOC)) { $__cfg[$__r['setting_key']] = $__r['setting_value']; }
        } catch (Throwable $e) {}
    }
    if (($__cfg['enabled'] ?? '1') !== '1') return;
    add_action('footer', function() use ($__cfg) {
        $imgUrl   = htmlspecialchars($__cfg['image_url'], ENT_QUOTES);
        $useEmoji = ($__cfg['use_emoji'] ?? '0') === '1';
        $emoji    = $__cfg['emoji'] ?: 'ðŸ›·ðŸŽ…ðŸŽ';
        $size     = max(50, min(400, (int)$__cfg['size']));
        $speed    = max(5, min(90, (float)$__cfg['speed']));
        $top      = max(0, min(80, (int)$__cfg['top']));
        $flip     = ($__cfg['flip'] ?? '0') === '1';
        ?>
<style>
.we-santa-sled{position:fixed;top:<?= $top ?>%;left:-400px;z-index:9997;
pointer-events:none;filter:drop-shadow(0 12px 20px rgba(0,0,0,0.45));
animation:weSledTravel <?= $speed ?>s linear infinite;white-space:nowrap}
.we-santa-sled img{width:<?= $size ?>px;height:auto;display:block;<?= $flip ? 'transform:scaleX(-1);' : '' ?>}
.we-santa-sled-emoji{font-size:<?= $size ?>px;display:block;<?= $flip ? 'transform:scaleX(-1);' : '' ?>}
@keyframes weSledTravel{from{left:-400px}to{left:110vw}}
</style>
<?php if ($useEmoji || $imgUrl === ''): ?>
<div class="we-santa-sled"><span class="we-santa-sled-emoji"><?= htmlspecialchars($emoji) ?></span></div>
<?php else: ?>
<div class="we-santa-sled"><img src="<?= $imgUrl ?>" alt="" loading="lazy"></div>
<?php endif; ?>
        <?php
    });
}

--- effect-snow ---
<?php
declare(strict_types=1);
if (!defined('WE_EFFECT_SNOW_LOADED')) {
    define('WE_EFFECT_SNOW_LOADED', true);
    $__cfg = ['enabled'=>'1','intensity'=>'60','speed'=>'6','size'=>'40','image_url'=>'https://cdn-icons-png.flaticon.com/512/642/642102.png','use_emoji'=>'0','emoji'=>'â„ï¸','opacity'=>'0.9'];
    if (isset($pdo) && $pdo instanceof PDO) {
        try {
            $__s = $pdo->prepare("SELECT setting_key,setting_value FROM plugin_settings WHERE plugin_slug=?");
            $__s->execute(['effect-snow']);
            while ($__r = $__s->fetch(PDO::FETCH_ASSOC)) { $__cfg[$__r['setting_key']] = $__r['setting_value']; }
        } catch (Throwable $e) {}
    }
    if (($__cfg['enabled'] ?? '1') !== '1') return;
    add_action('footer', function() use ($__cfg) {
        $intensity = max(1, min(300, (int)$__cfg['intensity']));
        $speed     = max(1, min(30, (float)$__cfg['speed']));
        $size      = max(10, min(150, (int)$__cfg['size']));
        $imgUrl    = htmlspecialchars($__cfg['image_url'], ENT_QUOTES);
        $useEmoji  = ($__cfg['use_emoji'] ?? '0') === '1';
        $emoji     = $__cfg['emoji'] ?: 'â„ï¸';
        $opacity   = max(0.1, min(1, (float)$__cfg['opacity']));
        ?>
<style>
.we-snow-container{position:fixed;inset:0;pointer-events:none;z-index:9998;overflow:hidden}
.we-snow-flake{position:absolute;top:-10%;opacity:<?= $opacity ?>;
user-select:none;animation:weSnowFall linear infinite,weSnowSway 4s ease-in-out infinite alternate}
.we-snow-flake img{width:100%;height:100%;display:block;filter:drop-shadow(0 0 6px rgba(255,255,255,0.5))}
@keyframes weSnowFall{to{transform:translateY(110vh)}}
@keyframes weSnowSway{from{margin-left:-25px}to{margin-left:25px}}
</style>
<div class="we-snow-container" id="weSnowContainer"></div>
<script>
(function(){var c=document.getElementById('weSnowContainer');if(!c)return;
var useEmoji=<?= $useEmoji ? 'true' : 'false' ?>;
var emoji=<?= json_encode($emoji) ?>;
var imgUrl=<?= json_encode($imgUrl) ?>;
for(var i=0;i<<?= $intensity ?>;i++){var f=document.createElement('div');f.className='we-snow-flake';
var s=(<?= $size ?>*(0.6+Math.random()*0.7));
f.style.width=s+'px';f.style.height=s+'px';
if(useEmoji||!imgUrl){f.textContent=emoji;f.style.fontSize=s+'px';f.style.color='#fff';}
else{var img=document.createElement('img');img.src=imgUrl;img.alt='';img.loading='lazy';f.appendChild(img);}
f.style.left=(Math.random()*100)+'%';
f.style.animationDelay=(Math.random()*<?= $speed ?>)+'s';
f.style.animationDuration=(<?= $speed ?>*(0.6+Math.random()*0.8))+'s';
c.appendChild(f);}})();

--- effect-train ---
<?php
declare(strict_types=1);
if (!defined('WE_EFFECT_TRAIN_LOADED')) {
    define('WE_EFFECT_TRAIN_LOADED', true);
    $__cfg = ['enabled'=>'1','image_url'=>'https://cdn-icons-png.flaticon.com/512/823/823241.png','use_emoji'=>'0','emoji'=>'ðŸš‚ðŸšƒðŸšƒðŸŽ','size'=>'90','speed'=>'25','bottom'=>'10'];
    if (isset($pdo) && $pdo instanceof PDO) {
        try {
            $__s = $pdo->prepare("SELECT setting_key,setting_value FROM plugin_settings WHERE plugin_slug=?");
            $__s->execute(['effect-train']);
            while ($__r = $__s->fetch(PDO::FETCH_ASSOC)) { $__cfg[$__r['setting_key']] = $__r['setting_value']; }
        } catch (Throwable $e) {}
    }
    if (($__cfg['enabled'] ?? '1') !== '1') return;
    add_action('footer', function() use ($__cfg) {
        $imgUrl   = htmlspecialchars($__cfg['image_url'], ENT_QUOTES);
        $useEmoji = ($__cfg['use_emoji'] ?? '0') === '1';
        $emoji    = $__cfg['emoji'] ?: 'ðŸš‚ðŸšƒðŸšƒðŸŽ';
        $size     = max(40, min(250, (int)$__cfg['size']));
        $speed    = max(5, min(90, (float)$__cfg['speed']));
        $bottom   = max(0, min(200, (int)$__cfg['bottom']));
        ?>
<style>
.we-xmas-train{position:fixed;bottom:<?= $bottom ?>px;left:-500px;z-index:9994;
pointer-events:none;white-space:nowrap;
animation:weTrainTravel <?= $speed ?>s linear infinite;
filter:drop-shadow(0 -4px 8px rgba(0,0,0,0.5))}
.we-xmas-train img{width:<?= $size ?>px;height:auto;display:block}
.we-xmas-train-emoji{font-size:<?= $size ?>px;display:block}
@keyframes weTrainTravel{from{left:-500px}to{left:110vw}}
</style>
<?php if ($useEmoji || $imgUrl === ''): ?>
<div class="we-xmas-train"><span class="we-xmas-train-emoji"><?= htmlspecialchars($emoji) ?></span></div>
<?php else: ?>
<div class="we-xmas-train"><img src="<?= $imgUrl ?>" alt="" loading="lazy"></div>
<?php endif; ?>
        <?php
    });
}

--- effect-witch ---
<?php
declare(strict_types=1);
if (!defined('WE_EFFECT_WITCH_LOADED')) {
    define('WE_EFFECT_WITCH_LOADED', true);
    $__cfg = ['enabled'=>'1','image_url'=>'https://cdn-icons-png.flaticon.com/512/3456/3456426.png','use_emoji'=>'0','emoji'=>'ðŸ§™â€â™€ï¸','size'=>'140','speed'=>'15','top'=>'25','flip'=>'1'];
    if (isset($pdo) && $pdo instanceof PDO) {
        try {
            $__s = $pdo->prepare("SELECT setting_key,setting_value FROM plugin_settings WHERE plugin_slug=?");
            $__s->execute(['effect-witch']);
            while ($__r = $__s->fetch(PDO::FETCH_ASSOC)) { $__cfg[$__r['setting_key']] = $__r['setting_value']; }
        } catch (Throwable $e) {}
    }
    if (($__cfg['enabled'] ?? '1') !== '1') return;
    add_action('footer', function() use ($__cfg) {
        $imgUrl   = htmlspecialchars($__cfg['image_url'], ENT_QUOTES);
        $useEmoji = ($__cfg['use_emoji'] ?? '0') === '1';
        $emoji    = $__cfg['emoji'] ?: 'ðŸ§™â€â™€ï¸';
        $size     = max(60, min(400, (int)$__cfg['size']));
        $speed    = max(4, min(60, (float)$__cfg['speed']));
        $top      = max(0, min(80, (int)$__cfg['top']));
        $flip     = ($__cfg['flip'] ?? '1') === '1';
        ?>
<style>
.we-witch{position:fixed;top:<?= $top ?>%;left:-350px;z-index:9997;pointer-events:none;
filter:drop-shadow(0 8px 15px rgba(0,0,0,0.5));
animation:weWitchFly <?= $speed ?>s linear infinite}
.we-witch img{width:<?= $size ?>px;height:auto;display:block;<?= $flip ? 'transform:scaleX(-1);' : '' ?>}
.we-witch-emoji{font-size:<?= $size ?>px;display:block;<?= $flip ? 'transform:scaleX(-1);' : '' ?>}
@keyframes weWitchFly{0%{left:-350px;transform:translateY(0)}
50%{transform:translateY(-50px)}
100%{left:110vw;transform:translateY(0)}}
</style>
<?php if ($useEmoji || $imgUrl === ''): ?>
<div class="we-witch"><span class="we-witch-emoji"><?= htmlspecialchars($emoji) ?></span></div>
<?php else: ?>
<div class="we-witch"><img src="<?= $imgUrl ?>" alt="" loading="lazy"></div>
<?php endif; ?>
        <?php
    });
}
