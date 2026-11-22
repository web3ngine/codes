WEB-ENGINE - PLUGIN SOURCE INSPECTION
Generated: 11/22/2026 03:43:29
Source: D:\Program Files\XAMPP\htdocs\webengine\plugins
Plugins: 13
==========================================================================================

# PLUGIN: effect-c4-bomb
==========================================================================================
FILES: 2
FILE: index.php | SIZE: 2448 bytes
FILE: plugin.json | SIZE: 220 bytes

------------------------------------------------------------------------------------------
SOURCE FILE: D:\Program Files\XAMPP\htdocs\webengine\plugins\effect-c4-bomb\index.php
------------------------------------------------------------------------------------------
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
<div class="we-c4"><img src="<?= $imgUrl ?>" alt="" loading="lazy"></div>
<?php endif; ?>
        <?php
    });
}

------------------------------------------------------------------------------------------
SOURCE FILE: D:\Program Files\XAMPP\htdocs\webengine\plugins\effect-c4-bomb\plugin.json
------------------------------------------------------------------------------------------
{
    "name": "CS 1.6 C4 Bomb",
    "version": "1.0.0",
    "author": "WebEngine",
    "description": "CS 1.6 C4 bomb (3D image with blinking LED)",
    "main": "index.php",
    "icon": "fa-bomb",
    "category": "fun"
}

VALIDATION:
PHP: No syntax errors detected in D:\Program Files\XAMPP\htdocs\webengine\plugins\effect-c4-bomb\index.php
JSON: VALID

# PLUGIN: effect-christmas-bg
==========================================================================================
FILES: 2
FILE: index.php | SIZE: 1745 bytes
FILE: plugin.json | SIZE: 262 bytes

------------------------------------------------------------------------------------------
SOURCE FILE: D:\Program Files\XAMPP\htdocs\webengine\plugins\effect-christmas-bg\index.php
------------------------------------------------------------------------------------------
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

------------------------------------------------------------------------------------------
SOURCE FILE: D:\Program Files\XAMPP\htdocs\webengine\plugins\effect-christmas-bg\plugin.json
------------------------------------------------------------------------------------------
{
    "name": "Christmas Background",
    "version": "1.0.0",
    "author": "WebEngine",
    "description": "Replace site background with a 4K Christmas image (presets + custom URL)",
    "main": "index.php",
    "icon": "fa-image",
    "category": "christmas"
}

VALIDATION:
PHP: No syntax errors detected in D:\Program Files\XAMPP\htdocs\webengine\plugins\effect-christmas-bg\index.php
JSON: VALID

# PLUGIN: effect-christmas-paws
==========================================================================================
FILES: 2
FILE: index.php | SIZE: 2562 bytes
FILE: plugin.json | SIZE: 219 bytes

------------------------------------------------------------------------------------------
SOURCE FILE: D:\Program Files\XAMPP\htdocs\webengine\plugins\effect-christmas-paws\index.php
------------------------------------------------------------------------------------------
<?php
declare(strict_types=1);
if (!defined('WE_EFFECT_XMAS_PAWS_LOADED')) {
    define('WE_EFFECT_XMAS_PAWS_LOADED', true);
    $__cfg = ['enabled'=>'1','image_url'=>'https://cdn-icons-png.flaticon.com/512/1998/1998592.png','use_emoji'=>'0','emoji'=>'ðŸ¾','size'=>'32','opacity'=>'0.65','lifetime'=>'2','min_dist'=>'70'];
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
        $emoji   = $__cfg['emoji'] ?: 'ðŸ¾';
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
document.body.appendChild(p);
setTimeout(function(){p.classList.add('fade-out')},life);
setTimeout(function(){p.remove()},life+1000);});})();
</script>
        <?php
    });
}

------------------------------------------------------------------------------------------
SOURCE FILE: D:\Program Files\XAMPP\htdocs\webengine\plugins\effect-christmas-paws\plugin.json
------------------------------------------------------------------------------------------
{
    "name": "Christmas Paws",
    "version": "1.0.0",
    "author": "WebEngine",
    "description": "3D paw prints follow the mouse cursor",
    "main": "index.php",
    "icon": "fa-paw",
    "category": "christmas"
}

VALIDATION:
PHP: No syntax errors detected in D:\Program Files\XAMPP\htdocs\webengine\plugins\effect-christmas-paws\index.php
JSON: VALID

# PLUGIN: effect-christmas-tree
==========================================================================================
FILES: 2
FILE: index.php | SIZE: 2010 bytes
FILE: plugin.json | SIZE: 222 bytes

------------------------------------------------------------------------------------------
SOURCE FILE: D:\Program Files\XAMPP\htdocs\webengine\plugins\effect-christmas-tree\index.php
------------------------------------------------------------------------------------------
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

------------------------------------------------------------------------------------------
SOURCE FILE: D:\Program Files\XAMPP\htdocs\webengine\plugins\effect-christmas-tree\plugin.json
------------------------------------------------------------------------------------------
{
    "name": "Christmas Tree",
    "version": "1.0.0",
    "author": "WebEngine",
    "description": "3D Christmas tree in bottom-left corner",
    "main": "index.php",
    "icon": "fa-tree",
    "category": "christmas"
}

VALIDATION:
PHP: No syntax errors detected in D:\Program Files\XAMPP\htdocs\webengine\plugins\effect-christmas-tree\index.php
JSON: VALID

# PLUGIN: effect-gifts
==========================================================================================
FILES: 2
FILE: index.php | SIZE: 2803 bytes
FILE: plugin.json | SIZE: 221 bytes

------------------------------------------------------------------------------------------
SOURCE FILE: D:\Program Files\XAMPP\htdocs\webengine\plugins\effect-gifts\index.php
------------------------------------------------------------------------------------------
<?php
declare(strict_types=1);
if (!defined('WE_EFFECT_GIFTS_LOADED')) {
    define('WE_EFFECT_GIFTS_LOADED', true);
    $__cfg = ['enabled'=>'1','intensity'=>'25','speed'=>'8','size'=>'70','image_urls'=>'','use_emoji'=>'0','emojis'=>'ðŸŽ,ðŸŽ€,ðŸ§¸'];
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
        if (empty($emojis)) $emojis = ['ðŸŽ'];
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
g.style.animationDelay=(Math.random()*<?= $speed ?>)+'s';
g.style.animationDuration=(<?= $speed ?>*(0.6+Math.random()*0.8))+'s';
c.appendChild(g);}})();
</script>
        <?php
    });
}

------------------------------------------------------------------------------------------
SOURCE FILE: D:\Program Files\XAMPP\htdocs\webengine\plugins\effect-gifts\plugin.json
------------------------------------------------------------------------------------------
{
    "name": "Falling Gifts",
    "version": "1.0.0",
    "author": "WebEngine",
    "description": "3D Christmas gifts falling from the sky",
    "main": "index.php",
    "icon": "fa-gift",
    "category": "christmas"
}

VALIDATION:
PHP: No syntax errors detected in D:\Program Files\XAMPP\htdocs\webengine\plugins\effect-gifts\index.php
JSON: VALID

# PLUGIN: effect-leaves
==========================================================================================
FILES: 2
FILE: index.php | SIZE: 2819 bytes
FILE: plugin.json | SIZE: 226 bytes

------------------------------------------------------------------------------------------
SOURCE FILE: D:\Program Files\XAMPP\htdocs\webengine\plugins\effect-leaves\index.php
------------------------------------------------------------------------------------------
<?php
declare(strict_types=1);
if (!defined('WE_EFFECT_LEAVES_LOADED')) {
    define('WE_EFFECT_LEAVES_LOADED', true);
    $__cfg = ['enabled'=>'1','intensity'=>'40','speed'=>'8','size'=>'50','image_urls'=>'','use_emoji'=>'0','emojis'=>'ðŸ‚,ðŸ,ðŸƒ'];
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
        if (empty($emojis)) $emojis = ['ðŸ‚'];
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
l.style.animationDelay=(Math.random()*<?= $speed ?>)+'s';
l.style.animationDuration=(<?= $speed ?>*(0.6+Math.random()*0.8))+'s';
c.appendChild(l);}})();
</script>
        <?php
    });
}

------------------------------------------------------------------------------------------
SOURCE FILE: D:\Program Files\XAMPP\htdocs\webengine\plugins\effect-leaves\plugin.json
------------------------------------------------------------------------------------------
{
    "name": "Falling Leaves",
    "version": "1.0.0",
    "author": "WebEngine",
    "description": "3D autumn leaves falling (multiple image URLs)",
    "main": "index.php",
    "icon": "fa-leaf",
    "category": "autumn"
}

VALIDATION:
PHP: No syntax errors detected in D:\Program Files\XAMPP\htdocs\webengine\plugins\effect-leaves\index.php
JSON: VALID

# PLUGIN: effect-lightning
==========================================================================================
FILES: 2
FILE: index.php | SIZE: 1800 bytes
FILE: plugin.json | SIZE: 225 bytes

------------------------------------------------------------------------------------------
SOURCE FILE: D:\Program Files\XAMPP\htdocs\webengine\plugins\effect-lightning\index.php
------------------------------------------------------------------------------------------
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

------------------------------------------------------------------------------------------
SOURCE FILE: D:\Program Files\XAMPP\htdocs\webengine\plugins\effect-lightning\plugin.json
------------------------------------------------------------------------------------------
{
    "name": "Lightning Effect",
    "version": "1.0.0",
    "author": "WebEngine",
    "description": "Random lightning flashes across the screen",
    "main": "index.php",
    "icon": "fa-bolt",
    "category": "weather"
}

VALIDATION:
PHP: No syntax errors detected in D:\Program Files\XAMPP\htdocs\webengine\plugins\effect-lightning\index.php
JSON: VALID

# PLUGIN: effect-rain
==========================================================================================
FILES: 2
FILE: index.php | SIZE: 2039 bytes
FILE: plugin.json | SIZE: 211 bytes

------------------------------------------------------------------------------------------
SOURCE FILE: D:\Program Files\XAMPP\htdocs\webengine\plugins\effect-rain\index.php
------------------------------------------------------------------------------------------
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

------------------------------------------------------------------------------------------
SOURCE FILE: D:\Program Files\XAMPP\htdocs\webengine\plugins\effect-rain\plugin.json
------------------------------------------------------------------------------------------
{
    "name": "Rain Effect",
    "version": "1.0.0",
    "author": "WebEngine",
    "description": "Animated falling rain drops",
    "main": "index.php",
    "icon": "fa-cloud-rain",
    "category": "weather"
}

VALIDATION:
PHP: No syntax errors detected in D:\Program Files\XAMPP\htdocs\webengine\plugins\effect-rain\index.php
JSON: VALID

# PLUGIN: effect-santa-hat
==========================================================================================
FILES: 2
FILE: index.php | SIZE: 2618 bytes
FILE: plugin.json | SIZE: 221 bytes

------------------------------------------------------------------------------------------
SOURCE FILE: D:\Program Files\XAMPP\htdocs\webengine\plugins\effect-santa-hat\index.php
------------------------------------------------------------------------------------------
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
setTimeout(add,500);setTimeout(add,1500);})();
</script>
        <?php
    });
}

------------------------------------------------------------------------------------------
SOURCE FILE: D:\Program Files\XAMPP\htdocs\webengine\plugins\effect-santa-hat\plugin.json
------------------------------------------------------------------------------------------
{
    "name": "Santa Hat on Logo",
    "version": "1.0.0",
    "author": "WebEngine",
    "description": "3D Santa hat on the site logo",
    "main": "index.php",
    "icon": "fa-hat-wizard",
    "category": "christmas"
}

VALIDATION:
PHP: No syntax errors detected in D:\Program Files\XAMPP\htdocs\webengine\plugins\effect-santa-hat\index.php
JSON: VALID

# PLUGIN: effect-santa-sled
==========================================================================================
FILES: 2
FILE: index.php | SIZE: 2055 bytes
FILE: plugin.json | SIZE: 226 bytes

------------------------------------------------------------------------------------------
SOURCE FILE: D:\Program Files\XAMPP\htdocs\webengine\plugins\effect-santa-sled\index.php
------------------------------------------------------------------------------------------
<?php
declare(strict_types=1);
if (!defined('WE_EFFECT_SANTA_SLED_LOADED')) {
    define('WE_EFFECT_SANTA_SLED_LOADED', true);
    $__cfg = ['enabled'=>'1','image_url'=>'https://cdn-icons-png.flaticon.com/512/3467/3467891.png','use_emoji'=>'0','emoji'=>'ðŸ›·ðŸŽ…ðŸŽ','size'=>'160','speed'=>'22','top'=>'15','flip'=>'0'];
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
        $emoji    = $__cfg['emoji'] ?: 'ðŸ›·ðŸŽ…ðŸŽ';
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

------------------------------------------------------------------------------------------
SOURCE FILE: D:\Program Files\XAMPP\htdocs\webengine\plugins\effect-santa-sled\plugin.json
------------------------------------------------------------------------------------------
{
    "name": "Santa Sled",
    "version": "1.0.0",
    "author": "WebEngine",
    "description": "3D Santa on a sled travelling across the site",
    "main": "index.php",
    "icon": "fa-sleigh",
    "category": "christmas"
}

VALIDATION:
PHP: No syntax errors detected in D:\Program Files\XAMPP\htdocs\webengine\plugins\effect-santa-sled\index.php
JSON: VALID

# PLUGIN: effect-snow
==========================================================================================
FILES: 2
FILE: index.php | SIZE: 2666 bytes
FILE: plugin.json | SIZE: 231 bytes

------------------------------------------------------------------------------------------
SOURCE FILE: D:\Program Files\XAMPP\htdocs\webengine\plugins\effect-snow\index.php
------------------------------------------------------------------------------------------
<?php
declare(strict_types=1);
if (!defined('WE_EFFECT_SNOW_LOADED')) {
    define('WE_EFFECT_SNOW_LOADED', true);
    $__cfg = ['enabled'=>'1','intensity'=>'60','speed'=>'6','size'=>'40','image_url'=>'https://cdn-icons-png.flaticon.com/512/642/642102.png','use_emoji'=>'0','emoji'=>'â„ï¸','opacity'=>'0.9'];
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
        $emoji     = $__cfg['emoji'] ?: 'â„ï¸';
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
</script>
        <?php
    });
}

------------------------------------------------------------------------------------------
SOURCE FILE: D:\Program Files\XAMPP\htdocs\webengine\plugins\effect-snow\plugin.json
------------------------------------------------------------------------------------------
{
    "name": "Snow Effect",
    "version": "1.0.0",
    "author": "WebEngine",
    "description": "Animated falling snowflakes (3D image support)",
    "main": "index.php",
    "icon": "fa-snowflake",
    "category": "christmas"
}

VALIDATION:
PHP: No syntax errors detected in D:\Program Files\XAMPP\htdocs\webengine\plugins\effect-snow\index.php
JSON: VALID

# PLUGIN: effect-train
==========================================================================================
FILES: 2
FILE: index.php | SIZE: 1907 bytes
FILE: plugin.json | SIZE: 218 bytes

------------------------------------------------------------------------------------------
SOURCE FILE: D:\Program Files\XAMPP\htdocs\webengine\plugins\effect-train\index.php
------------------------------------------------------------------------------------------
<?php
declare(strict_types=1);
if (!defined('WE_EFFECT_TRAIN_LOADED')) {
    define('WE_EFFECT_TRAIN_LOADED', true);
    $__cfg = ['enabled'=>'1','image_url'=>'https://cdn-icons-png.flaticon.com/512/823/823241.png','use_emoji'=>'0','emoji'=>'ðŸš‚ðŸšƒðŸšƒðŸŽ','size'=>'90','speed'=>'25','bottom'=>'10'];
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
        $emoji    = $__cfg['emoji'] ?: 'ðŸš‚ðŸšƒðŸšƒðŸŽ';
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

------------------------------------------------------------------------------------------
SOURCE FILE: D:\Program Files\XAMPP\htdocs\webengine\plugins\effect-train\plugin.json
------------------------------------------------------------------------------------------
{
    "name": "Christmas Train",
    "version": "1.0.0",
    "author": "WebEngine",
    "description": "3D train travels along the footer",
    "main": "index.php",
    "icon": "fa-train",
    "category": "christmas"
}

VALIDATION:
PHP: No syntax errors detected in D:\Program Files\XAMPP\htdocs\webengine\plugins\effect-train\index.php
JSON: VALID

# PLUGIN: effect-witch
==========================================================================================
FILES: 2
FILE: index.php | SIZE: 2071 bytes
FILE: plugin.json | SIZE: 230 bytes

------------------------------------------------------------------------------------------
SOURCE FILE: D:\Program Files\XAMPP\htdocs\webengine\plugins\effect-witch\index.php
------------------------------------------------------------------------------------------
<?php
declare(strict_types=1);
if (!defined('WE_EFFECT_WITCH_LOADED')) {
    define('WE_EFFECT_WITCH_LOADED', true);
    $__cfg = ['enabled'=>'1','image_url'=>'https://cdn-icons-png.flaticon.com/512/3456/3456426.png','use_emoji'=>'0','emoji'=>'ðŸ§™â€â™€ï¸','size'=>'140','speed'=>'15','top'=>'25','flip'=>'1'];
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
        $emoji    = $__cfg['emoji'] ?: 'ðŸ§™â€â™€ï¸';
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

------------------------------------------------------------------------------------------
SOURCE FILE: D:\Program Files\XAMPP\htdocs\webengine\plugins\effect-witch\plugin.json
------------------------------------------------------------------------------------------
{
    "name": "Witch on Broom",
    "version": "1.0.0",
    "author": "WebEngine",
    "description": "3D witch flies across the site on a broom",
    "main": "index.php",
    "icon": "fa-hat-wizard",
    "category": "halloween"
}

VALIDATION:
PHP: No syntax errors detected in D:\Program Files\XAMPP\htdocs\webengine\plugins\effect-witch\index.php
JSON: VALID
