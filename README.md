WEB-ENGINE PLUGIN SOURCE REVIEW

========== effect-c4-bomb ==========

--- plugin.json ---
{
    "name": "CS 1.6 C4 Bomb",
    "version": "1.0.0",
    "author": "WebEngine",
    "description": "CS 1.6 C4 bomb (3D image with blinking LED)",
    "main": "index.php",
    "icon": "fa-bomb",
    "category": "fun"
}

--- index.php ---
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

========== effect-christmas-bg ==========

--- plugin.json ---
{
    "name": "Christmas Background",
    "version": "1.0.0",
    "author": "WebEngine",
    "description": "Replace site background with a 4K Christmas image (presets + custom URL)",
    "main": "index.php",
    "icon": "fa-image",
    "category": "christmas"
}

--- index.php ---
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

========== effect-christmas-paws ==========

--- plugin.json ---
{
    "name": "Christmas Paws",
    "version": "1.0.0",
    "author": "WebEngine",
    "description": "3D paw prints follow the mouse cursor",
    "main": "index.php",
    "icon": "fa-paw",
    "category": "christmas"
}

--- index.php ---
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

========== effect-christmas-tree ==========

--- plugin.json ---
{
    "name": "Christmas Tree",
    "version": "1.0.0",
    "author": "WebEngine",
    "description": "3D Christmas tree in bottom-left corner",
    "main": "index.php",
    "icon": "fa-tree",
    "category": "christmas"
}

--- index.php ---
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

========== effect-gifts ==========

--- plugin.json ---
{
    "name": "Falling Gifts",
    "version": "1.0.0",
    "author": "WebEngine",
    "description": "3D Christmas gifts falling from the sky",
    "main": "index.php",
    "icon": "fa-gift",
    "category": "christmas"
}

--- index.php ---
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

========== effect-leaves ==========

--- plugin.json ---
{
    "name": "Falling Leaves",
    "version": "1.0.0",
    "author": "WebEngine",
    "description": "3D autumn leaves falling (multiple image URLs)",
    "main": "index.php",
    "icon": "fa-leaf",
    "category": "autumn"
}

--- index.php ---
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

========== effect-lightning ==========

--- plugin.json ---
{
    "name": "Lightning Effect",
    "version": "1.0.0",
    "author": "WebEngine",
    "description": "Random lightning flashes across the screen",
    "main": "index.php",
    "icon": "fa-bolt",
    "category": "weather"
}

--- index.php ---
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

========== effect-rain ==========

--- plugin.json ---
{
    "name": "Rain Effect",
    "version": "1.0.0",
    "author": "WebEngine",
    "description": "Animated falling rain drops",
    "main": "index.php",
    "icon": "fa-cloud-rain",
    "category": "weather"
}

--- index.php ---
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

========== effect-santa-hat ==========

--- plugin.json ---
{
    "name": "Santa Hat on Logo",
    "version": "1.0.0",
    "author": "WebEngine",
    "description": "3D Santa hat on the site logo",
    "main": "index.php",
    "icon": "fa-hat-wizard",
    "category": "christmas"
}

--- index.php ---
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

========== effect-santa-sled ==========

--- plugin.json ---
{
    "name": "Santa Sled",
    "version": "1.0.0",
    "author": "WebEngine",
    "description": "3D Santa on a sled travelling across the site",
    "main": "index.php",
    "icon": "fa-sleigh",
    "category": "christmas"
}

--- index.php ---
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

========== effect-snow ==========

--- plugin.json ---
{
    "name": "Snow Effect",
    "version": "1.0.0",
    "author": "WebEngine",
    "description": "Animated falling snowflakes (3D image support)",
    "main": "index.php",
    "icon": "fa-snowflake",
    "category": "christmas"
}

--- index.php ---
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

========== effect-train ==========

--- plugin.json ---
{
    "name": "Christmas Train",
    "version": "1.0.0",
    "author": "WebEngine",
    "description": "3D train travels along the footer",
    "main": "index.php",
    "icon": "fa-train",
    "category": "christmas"
}

--- index.php ---
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

========== effect-witch ==========

--- plugin.json ---
{
    "name": "Witch on Broom",
    "version": "1.0.0",
    "author": "WebEngine",
    "description": "3D witch flies across the site on a broom",
    "main": "index.php",
    "icon": "fa-hat-wizard",
    "category": "halloween"
}

--- index.php ---
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

========== PLUGIN SETTINGS DEFINITIONS ==========
<?php
declare(strict_types=1);
error_reporting(E_ALL);
ini_set('display_errors', '0');
ini_set('log_errors', '1');
set_time_limit(600);

/* ==========================================================
   WEBENGINE ADMIN PANEL
   File: adm/plugins.php
   Purpose: Plugin manager + Auto Effect Generator (3D/4K images)
   ========================================================== */

require_once __DIR__ . '/../config/database.php';

if (session_status() === PHP_SESSION_NONE) {
    session_start();
}

/* ==========================================================
   AUTHENTICATION
   ========================================================== */

$adminId = $_SESSION['admin_id'] ?? null;

if (!$adminId) {
    header('Location: adm/login.php');
    exit;
}

$stmt = $pdo->prepare("SELECT id, username, avatar, role FROM users WHERE id = ? LIMIT 1");
$stmt->execute([$adminId]);
$admin = $stmt->fetch(PDO::FETCH_ASSOC);

if (!$admin || $admin['role'] !== 'admin') {
    unset($_SESSION['admin_id'], $_SESSION['admin_username'], $_SESSION['admin_email'],
          $_SESSION['admin_role'], $_SESSION['admin_login_at']);
    header('Location: adm/login.php');
    exit;
}

$username = htmlspecialchars($admin['username'] ?? 'Admin', ENT_QUOTES, 'UTF-8');

/* ==========================================================
   AVATAR
   ========================================================== */

$avatarFile = trim((string)($admin['avatar'] ?? ''));

if ($avatarFile !== '') {
    $avatarFile = ltrim($avatarFile, '/');
    if (str_starts_with($avatarFile, 'uploads/avatars/')) {
        $avatarFile = substr($avatarFile, strlen('uploads/avatars/'));
    } elseif (str_starts_with($avatarFile, 'uploads/')) {
        $avatarFile = substr($avatarFile, strlen('uploads/'));
    }
}

$avatarDiskPath = __DIR__ . '/../uploads/avatars/' . $avatarFile;
$avatarWebPath  = '../uploads/avatars/' . $avatarFile;

if ($avatarFile === '' || !is_file($avatarDiskPath)) {
    $avatarWebPath = '../uploads/avatars/default.png';
}
$avatar = htmlspecialchars($avatarWebPath, ENT_QUOTES, 'UTF-8');

/* ==========================================================
   PATHS
   ========================================================== */

$basePath    = realpath(__DIR__ . '/..');
$pluginsDir  = $basePath . '/plugins';

if (!is_dir($pluginsDir)) {
    @mkdir($pluginsDir, 0755, true);
}

/* ==========================================================
   AUTO-CREATE TABLES
   ========================================================== */

try {
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

    $pdo->exec("
        CREATE TABLE IF NOT EXISTS `plugin_settings` (
            `id` INT(11) NOT NULL AUTO_INCREMENT,
            `plugin_slug` VARCHAR(100) NOT NULL,
            `setting_key` VARCHAR(100) NOT NULL,
            `setting_value` TEXT DEFAULT NULL,
            `updated_at` DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
            PRIMARY KEY (`id`),
            UNIQUE KEY `uniq_plugin_setting` (`plugin_slug`, `setting_key`),
            KEY `idx_plugin` (`plugin_slug`)
        ) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci
    ");
} catch (Throwable $e) {}

/* ==========================================================
   HELPERS
   ========================================================== */

function timeAgo(string $datetime): string {
    if (empty($datetime)) return '-';
    $diff = time() - strtotime($datetime);
    if ($diff < 60)       return 'just now';
    if ($diff < 3600)     return floor($diff / 60) . ' min ago';
    if ($diff < 86400)    return floor($diff / 3600) . 'h ago';
    if ($diff < 604800)   return floor($diff / 86400) . 'd ago';
    if ($diff < 2592000)  return floor($diff / 604800) . 'w ago';
    return floor($diff / 2592000) . 'mo ago';
}

function registerPlugin(PDO $pdo, array $meta, array $settings = []): array {
    $slug = $meta['slug'];

    try {
        $stmt = $pdo->prepare("SELECT id FROM plugins WHERE slug = ? LIMIT 1");
        $stmt->execute([$slug]);

        if ($stmt->fetchColumn()) {
            return ['status' => 'skip', 'name' => $meta['name']];
        }

        $pdo->prepare("
            INSERT INTO plugins
                (slug, name, description, version, author, icon, category, main_file, status)
            VALUES (?, ?, ?, ?, ?, ?, ?, ?, 'inactive')
        ")->execute([
            $slug,
            $meta['name'],
            $meta['description'] ?? '',
            $meta['version'] ?? '1.0.0',
            $meta['author'] ?? 'WebEngine',
            $meta['icon'] ?? 'fa-puzzle-piece',
            $meta['category'] ?? 'general',
            $meta['main'] ?? 'index.php',
        ]);

        foreach ($settings as $k => $v) {
            if (is_array($v)) continue;
            $pdo->prepare("
                INSERT INTO plugin_settings (plugin_slug, setting_key, setting_value)
                VALUES (?, ?, ?)
                ON DUPLICATE KEY UPDATE setting_value = VALUES(setting_value)
            ")->execute([$slug, $k, (string)$v]);
        }

        return ['status' => 'created', 'name' => $meta['name']];
    } catch (Throwable $e) {
        return ['status' => 'error', 'name' => $meta['name'], 'error' => $e->getMessage()];
    }
}

/* ==========================================================
   EFFECT PLUGINS - 3D/4K IMAGE VERSION
   ========================================================== */

function getEffectPlugins(): array {
    $plugins = [];

    /* ============================================================
       1. RAIN
       ============================================================ */
    $plugins['effect-rain'] = [
        'meta' => [
            'slug' => 'effect-rain', 'name' => 'Rain Effect',
            'description' => 'Animated falling rain drops',
            'version' => '1.0.0', 'author' => 'WebEngine',
            'icon' => 'fa-cloud-rain', 'category' => 'weather', 'main' => 'index.php',
        ],
        'settings' => [
            'enabled'    => '1',
            'intensity'  => '120',
            'speed'      => '2',
            'color'      => '#93c5fd',
            'opacity'    => '0.65',
            'drop_width' => '2',
            'drop_height'=> '90',
        ],
        'code' => <<<'PHP'
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
PHP
    ];

    /* ============================================================
       2. SNOW - 3D IMAGE
       ============================================================ */
    $plugins['effect-snow'] = [
        'meta' => [
            'slug' => 'effect-snow', 'name' => 'Snow Effect',
            'description' => 'Animated falling snowflakes (3D image support)',
            'version' => '1.0.0', 'author' => 'WebEngine',
            'icon' => 'fa-snowflake', 'category' => 'christmas', 'main' => 'index.php',
        ],
        'settings' => [
            'enabled'    => '1',
            'intensity'  => '60',
            'speed'      => '6',
            'size'       => '40',
            'image_url'  => 'https://cdn-icons-png.flaticon.com/512/642/642102.png',
            'use_emoji'  => '0',
            'emoji'      => 'â„ï¸',
            'opacity'    => '0.9',
        ],
        'code' => <<<'PHP'
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
PHP
    ];

    /* ============================================================
       3. SANTA HAT - 3D IMAGE
       ============================================================ */
    $plugins['effect-santa-hat'] = [
        'meta' => [
            'slug' => 'effect-santa-hat', 'name' => 'Santa Hat on Logo',
            'description' => '3D Santa hat on the site logo',
            'version' => '1.0.0', 'author' => 'WebEngine',
            'icon' => 'fa-hat-wizard', 'category' => 'christmas', 'main' => 'index.php',
        ],
        'settings' => [
            'enabled'   => '1',
            'image_url' => 'https://cdn-icons-png.flaticon.com/512/6145/6145967.png',
            'use_emoji' => '0',
            'emoji'     => 'ðŸŽ…',
            'size'      => '80',
            'rotate'    => '-30',
            'top'       => '-30',
            'left'      => '20',
        ],
        'code' => <<<'PHP'
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
PHP
    ];

    /* ============================================================
       4. SANTA SLED - 3D IMAGE
       ============================================================ */
    $plugins['effect-santa-sled'] = [
        'meta' => [
            'slug' => 'effect-santa-sled', 'name' => 'Santa Sled',
            'description' => '3D Santa on a sled travelling across the site',
            'version' => '1.0.0', 'author' => 'WebEngine',
            'icon' => 'fa-sleigh', 'category' => 'christmas', 'main' => 'index.php',
        ],
        'settings' => [
            'enabled'   => '1',
            'image_url' => 'https://cdn-icons-png.flaticon.com/512/3467/3467891.png',
            'use_emoji' => '0',
            'emoji'     => 'ðŸ›·ðŸŽ…ðŸŽ',
            'size'      => '160',
            'speed'     => '22',
            'top'       => '15',
            'flip'      => '0',
        ],
        'code' => <<<'PHP'
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
PHP
    ];

    /* ============================================================
       5. CHRISTMAS BACKGROUND - 4K IMAGE + PRESETS
       ============================================================ */
    $plugins['effect-christmas-bg'] = [
        'meta' => [
            'slug' => 'effect-christmas-bg', 'name' => 'Christmas Background',
            'description' => 'Replace site background with a 4K Christmas image (presets + custom URL)',
            'version' => '1.0.0', 'author' => 'WebEngine',
            'icon' => 'fa-image', 'category' => 'christmas', 'main' => 'index.php',
        ],
        'settings' => [
            'enabled'    => '1',
            'image_url'  => 'https://images.unsplash.com/photo-1543589077-47d81606c1bf?w=2560&q=90',
            'opacity'    => '0.5',
            'blur'       => '0',
            'fixed'      => '1',
            'darken'     => '0.35',
        ],
        'code' => <<<'PHP'
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
PHP
    ];

    /* ============================================================
       6. CHRISTMAS PAWS - 3D IMAGE
       ============================================================ */
    $plugins['effect-christmas-paws'] = [
        'meta' => [
            'slug' => 'effect-christmas-paws', 'name' => 'Christmas Paws',
            'description' => '3D paw prints follow the mouse cursor',
            'version' => '1.0.0', 'author' => 'WebEngine',
            'icon' => 'fa-paw', 'category' => 'christmas', 'main' => 'index.php',
        ],
        'settings' => [
            'enabled'   => '1',
            'image_url' => 'https://cdn-icons-png.flaticon.com/512/1998/1998592.png',
            'use_emoji' => '0',
            'emoji'     => 'ðŸ¾',
            'size'      => '32',
            'opacity'   => '0.65',
            'lifetime'  => '2',
            'min_dist'  => '70',
        ],
        'code' => <<<'PHP'
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
PHP
    ];

    /* ============================================================
       7. CHRISTMAS TREE - 3D IMAGE
       ============================================================ */
    $plugins['effect-christmas-tree'] = [
        'meta' => [
            'slug' => 'effect-christmas-tree', 'name' => 'Christmas Tree',
            'description' => '3D Christmas tree in bottom-left corner',
            'version' => '1.0.0', 'author' => 'WebEngine',
            'icon' => 'fa-tree', 'category' => 'christmas', 'main' => 'index.php',
        ],
        'settings' => [
            'enabled'   => '1',
            'image_url' => 'https://cdn-icons-png.flaticon.com/512/2941/2941733.png',
            'use_emoji' => '0',
            'emoji'     => 'ðŸŽ„',
            'size'      => '140',
            'bottom'    => '20',
            'left'      => '20',
            'sway'      => '1',
        ],
        'code' => <<<'PHP'
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
PHP
    ];

    /* ============================================================
       8. TRAIN - 3D IMAGE
       ============================================================ */
    $plugins['effect-train'] = [
        'meta' => [
            'slug' => 'effect-train', 'name' => 'Christmas Train',
            'description' => '3D train travels along the footer',
            'version' => '1.0.0', 'author' => 'WebEngine',
            'icon' => 'fa-train', 'category' => 'christmas', 'main' => 'index.php',
        ],
        'settings' => [
            'enabled'   => '1',
            'image_url' => 'https://cdn-icons-png.flaticon.com/512/823/823241.png',
            'use_emoji' => '0',
            'emoji'     => 'ðŸš‚ðŸšƒðŸšƒðŸŽ',
            'size'      => '90',
            'speed'     => '25',
            'bottom'    => '10',
        ],
        'code' => <<<'PHP'
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
PHP
    ];

    /* ============================================================
       9. LIGHTNING
       ============================================================ */
    $plugins['effect-lightning'] = [
        'meta' => [
            'slug' => 'effect-lightning', 'name' => 'Lightning Effect',
            'description' => 'Random lightning flashes across the screen',
            'version' => '1.0.0', 'author' => 'WebEngine',
            'icon' => 'fa-bolt', 'category' => 'weather', 'main' => 'index.php',
        ],
        'settings' => [
            'enabled'  => '1',
            'interval' => '7',
            'duration' => '0.4',
            'color'    => '#ffffff',
            'opacity'  => '0.85',
        ],
        'code' => <<<'PHP'
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
PHP
    ];

    /* ============================================================
       10. LEAVES - 3D IMAGES
       ============================================================ */
    $plugins['effect-leaves'] = [
        'meta' => [
            'slug' => 'effect-leaves', 'name' => 'Falling Leaves',
            'description' => '3D autumn leaves falling (multiple image URLs)',
            'version' => '1.0.0', 'author' => 'WebEngine',
            'icon' => 'fa-leaf', 'category' => 'autumn', 'main' => 'index.php',
        ],
        'settings' => [
            'enabled'      => '1',
            'intensity'    => '40',
            'speed'        => '8',
            'size'         => '50',
            'image_urls'   => 'https://cdn-icons-png.flaticon.com/512/2913/2913465.png,https://cdn-icons-png.flaticon.com/512/2913/2913469.png,https://cdn-icons-png.flaticon.com/512/2913/2913471.png',
            'use_emoji'    => '0',
            'emojis'       => 'ðŸ‚,ðŸ,ðŸƒ',
        ],
        'code' => <<<'PHP'
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
PHP
    ];

    /* ============================================================
       11. GIFTS - 3D IMAGES
       ============================================================ */
    $plugins['effect-gifts'] = [
        'meta' => [
            'slug' => 'effect-gifts', 'name' => 'Falling Gifts',
            'description' => '3D Christmas gifts falling from the sky',
            'version' => '1.0.0', 'author' => 'WebEngine',
            'icon' => 'fa-gift', 'category' => 'christmas', 'main' => 'index.php',
        ],
        'settings' => [
            'enabled'   => '1',
            'intensity' => '25',
            'speed'     => '8',
            'size'      => '70',
            'image_urls'=> 'https://cdn-icons-png.flaticon.com/512/4213/4213179.png,https://cdn-icons-png.flaticon.com/512/4213/4213188.png,https://cdn-icons-png.flaticon.com/512/4213/4213190.png',
            'use_emoji' => '0',
            'emojis'    => 'ðŸŽ,ðŸŽ€,ðŸ§¸',
        ],
        'code' => <<<'PHP'
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
PHP
    ];

    /* ============================================================
       12. WITCH - 3D IMAGE
       ============================================================ */
    $plugins['effect-witch'] = [
        'meta' => [
            'slug' => 'effect-witch', 'name' => 'Witch on Broom',
            'description' => '3D witch flies across the site on a broom',
            'version' => '1.0.0', 'author' => 'WebEngine',
            'icon' => 'fa-hat-wizard', 'category' => 'halloween', 'main' => 'index.php',
        ],
        'settings' => [
            'enabled'   => '1',
            'image_url' => 'https://cdn-icons-png.flaticon.com/512/3456/3456426.png',
            'use_emoji' => '0',
            'emoji'     => 'ðŸ§™â€â™€ï¸',
            'size'      => '140',
            'speed'     => '15',
            'top'       => '25',
            'flip'      => '1',
        ],
        'code' => <<<'PHP'
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
PHP
    ];

    /* ============================================================
       13. C4 BOMB - 3D IMAGE
       ============================================================ */
    $plugins['effect-c4-bomb'] = [
        'meta' => [
            'slug' => 'effect-c4-bomb', 'name' => 'CS 1.6 C4 Bomb',
            'description' => 'CS 1.6 C4 bomb (3D image with blinking LED)',
            'version' => '1.0.0', 'author' => 'WebEngine',
            'icon' => 'fa-bomb', 'category' => 'fun', 'main' => 'index.php',
        ],
        'settings' => [
            'enabled'   => '1',
            'image_url' => 'https://cdn-icons-png.flaticon.com/512/3079/3079508.png',
            'use_emoji' => '0',
            'emoji'     => 'ðŸ’£',
            'size'      => '100',
            'position'  => 'br',
            'blink'     => '1',
            'glow_color'=> '#ff0000',
        ],
        'code' => <<<'PHP'
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
PHP
    ];

    return $plugins;
}

/* ==========================================================
   GENERATE ALL EFFECTS
   ========================================================== */

function generateAllEffects(PDO $pdo, string $pluginsDir): array {
    $effects = getEffectPlugins();
    $results = ['created' => 0, 'skipped' => 0, 'errors' => [], 'files_written' => 0];

    foreach ($effects as $slug => $data) {
        $meta = $data['meta'];
        $code = $data['code'];
        $settings = $data['settings'];

        $pluginDir = $pluginsDir . '/' . $slug;

        if (!is_dir($pluginDir)) {
            if (!@mkdir($pluginDir, 0755, true)) {
                $results['errors'][] = "Cannot create folder for $slug";
                continue;
            }
        }

        $manifest = [
            'name'        => $meta['name'],
            'version'     => $meta['version'],
            'author'      => $meta['author'],
            'description' => $meta['description'],
            'main'        => $meta['main'],
            'icon'        => $meta['icon'],
            'category'    => $meta['category'],
        ];

        if (@file_put_contents(
            $pluginDir . '/plugin.json',
            json_encode($manifest, JSON_PRETTY_PRINT | JSON_UNESCAPED_UNICODE)
        ) !== false) {
            $results['files_written']++;
        }

        if (@file_put_contents($pluginDir . '/index.php', $code) !== false) {
            $results['files_written']++;
        }

        $result = registerPlugin($pdo, $meta, $settings);

        if ($result['status'] === 'created') $results['created']++;
        elseif ($result['status'] === 'skip') $results['skipped']++;
        else $results['errors'][] = $result['error'] ?? 'Unknown error';
    }

    return $results;
}

/* ==========================================================
   AJAX TOGGLE HANDLER (NO PAGE RELOAD)
   ========================================================== */

if ($_SERVER['REQUEST_METHOD'] === 'POST' && ($_POST['action'] ?? '') === 'ajax_toggle') {
    header('Content-Type: application/json; charset=utf-8');

    $id = (int)($_POST['id'] ?? 0);
    if ($id <= 0) {
        echo json_encode(['ok' => false, 'message' => 'Invalid plugin ID.']);
        exit;
    }

    try {
        $stmt = $pdo->prepare("SELECT * FROM plugins WHERE id = ? LIMIT 1");
        $stmt->execute([$id]);
        $p = $stmt->fetch(PDO::FETCH_ASSOC);

        if (!$p) {
            echo json_encode(['ok' => false, 'message' => 'Plugin not found.']);
            exit;
        }

        if ((int)$p['is_required'] === 1 && $p['status'] === 'active') {
            echo json_encode(['ok' => false, 'message' => 'Cannot deactivate required plugin.']);
            exit;
        }

        $newStatus   = $p['status'] === 'active' ? 'inactive' : 'active';
        $activatedAt = $newStatus === 'active' ? date('Y-m-d H:i:s') : null;

        $pdo->prepare("UPDATE plugins SET status = ?, activated_at = ? WHERE id = ?")
            ->execute([$newStatus, $activatedAt, $id]);

        echo json_encode([
            'ok'      => true,
            'status'  => $newStatus,
            'message' => "Plugin '{$p['name']}' " . ($newStatus === 'active' ? 'activated.' : 'deactivated.'),
        ]);
    } catch (Throwable $e) {
        echo json_encode(['ok' => false, 'message' => 'Error: ' . $e->getMessage()]);
    }
    exit;
}

/* ==========================================================
   HANDLE ACTIONS (NON-AJAX)
   ========================================================== */

$flash = null;

if ($_SERVER['REQUEST_METHOD'] === 'POST') {

    $action = $_POST['action'] ?? '';

    if ($action === 'generate_all_effects') {
        $res = generateAllEffects($pdo, $pluginsDir);
        $msg = "Created: {$res['created']} - Skipped: {$res['skipped']} - Files: {$res['files_written']}";
        if (!empty($res['errors'])) $msg .= ' | Errors: ' . implode(', ', array_slice($res['errors'], 0, 3));
        $flash = ['type' => $res['created'] > 0 ? 'success' : 'info', 'message' => $msg];
    }

    elseif ($action === 'delete') {
        $id = (int)($_POST['id'] ?? 0);
        $removeFiles = isset($_POST['remove_files']) && $_POST['remove_files'] === '1';

        if ($id > 0) {
            $stmt = $pdo->prepare("SELECT * FROM plugins WHERE id = ? LIMIT 1");
            $stmt->execute([$id]);
            $p = $stmt->fetch(PDO::FETCH_ASSOC);

            if ($p) {
                if ((int)$p['is_required'] === 1) {
                    $flash = ['type' => 'error', 'message' => 'Cannot delete required plugin.'];
                } else {
                    $pdo->prepare("DELETE FROM plugin_settings WHERE plugin_slug = ?")->execute([$p['slug']]);
                    $pdo->prepare("DELETE FROM plugins WHERE id = ?")->execute([$id]);

                    if ($removeFiles) {
                        $path = $pluginsDir . '/' . $p['slug'];
                        if (is_dir($path)) {
                            $it = new RecursiveIteratorIterator(
                                new RecursiveDirectoryIterator($path, RecursiveDirectoryIterator::SKIP_DOTS),
                                RecursiveIteratorIterator::CHILD_FIRST
                            );
                            foreach ($it as $f) {
                                if ($f->isDir()) @rmdir($f->getPathname());
                                else @unlink($f->getPathname());
                            }
                            @rmdir($path);
                        }
                    }
                    $flash = ['type' => 'success', 'message' => "Plugin '{$p['name']}' deleted."];
                }
            }
        }
    }

    elseif ($action === 'delete_all_effects') {
        $effects = getEffectPlugins();
        $deleted = 0;
        foreach (array_keys($effects) as $slug) {
            try {
                $pdo->prepare("DELETE FROM plugin_settings WHERE plugin_slug = ?")->execute([$slug]);
                $stmt = $pdo->prepare("DELETE FROM plugins WHERE slug = ?");
                $stmt->execute([$slug]);
                $deleted += $stmt->rowCount();

                $path = $pluginsDir . '/' . $slug;
                if (is_dir($path)) {
                    $it = new RecursiveIteratorIterator(
                        new RecursiveDirectoryIterator($path, RecursiveDirectoryIterator::SKIP_DOTS),
                        RecursiveIteratorIterator::CHILD_FIRST
                    );
                    foreach ($it as $f) {
                        if ($f->isDir()) @rmdir($f->getPathname());
                        else @unlink($f->getPathname());
                    }
                    @rmdir($path);
                }
            } catch (Throwable $e) {}
        }
        $flash = ['type' => 'success', 'message' => "Deleted $deleted effect plugin(s)."];
    }

    elseif ($action === 'save_settings') {
        $slug = trim((string)($_POST['slug'] ?? ''));
        $settings = $_POST['settings'] ?? [];

        if ($slug !== '' && is_array($settings)) {
            foreach ($settings as $k => $v) {
                $k = preg_replace('/[^a-z0-9_\-]/i', '', (string)$k);
                if ($k === '') continue;
                try {
                    $pdo->prepare("
                        INSERT INTO plugin_settings (plugin_slug, setting_key, setting_value)
                        VALUES (?, ?, ?)
                        ON DUPLICATE KEY UPDATE setting_value = VALUES(setting_value)
                    ")->execute([$slug, $k, (string)$v]);
                } catch (Throwable $e) {}
            }
            $flash = ['type' => 'success', 'message' => 'Settings saved.'];
        }
    }

    if ($flash) $_SESSION['plugins_flash'] = $flash;
    $redirectUrl = $_SERVER['PHP_SELF'];
    if (!empty($_GET)) $redirectUrl .= '?' . http_build_query($_GET);
    header('Location: ' . $redirectUrl);
    exit;
}

if (!empty($_SESSION['plugins_flash'])) {
    $flash = $_SESSION['plugins_flash'];
    unset($_SESSION['plugins_flash']);
}

/* ==========================================================
   LOAD PLUGINS
   ========================================================== */

$filterStatus = $_GET['status'] ?? '';
$search = trim((string)($_GET['search'] ?? ''));

$where = ['1 = 1'];
$params = [];

if (in_array($filterStatus, ['active','inactive'], true)) {
    $where[] = 'status = ?';
    $params[] = $filterStatus;
}

if ($search !== '') {
    $where[] = '(name LIKE ? OR slug LIKE ? OR description LIKE ?)';
    $params[] = "%$search%";
    $params[] = "%$search%";
    $params[] = "%$search%";
}

$whereSql = 'WHERE ' . implode(' AND ', $where);

$plugins = [];
try {
    $stmt = $pdo->prepare("SELECT * FROM plugins $whereSql ORDER BY status DESC, name ASC");
    $stmt->execute($params);
    $plugins = $stmt->fetchAll(PDO::FETCH_ASSOC);
} catch (Throwable $e) {}

$statTotal    = 0;
$statActive   = 0;
$statInactive = 0;
$statEffects  = 0;

try {
    $statTotal    = (int)$pdo->query("SELECT COUNT(*) FROM plugins")->fetchColumn();
    $statActive   = (int)$pdo->query("SELECT COUNT(*) FROM plugins WHERE status = 'active'")->fetchColumn();
    $statInactive = (int)$pdo->query("SELECT COUNT(*) FROM plugins WHERE status = 'inactive'")->fetchColumn();
    $statEffects  = (int)$pdo->query("SELECT COUNT(*) FROM plugins WHERE slug LIKE 'effect-%'")->fetchColumn();
} catch (Throwable $e) {}

$viewPlugin = null;
$viewSettings = [];

if (!empty($_GET['view'])) {
    $id = (int)$_GET['view'];
    try {
        $stmt = $pdo->prepare("SELECT * FROM plugins WHERE id = ? LIMIT 1");
        $stmt->execute([$id]);
        $viewPlugin = $stmt->fetch(PDO::FETCH_ASSOC);

        if ($viewPlugin) {
            $stmt = $pdo->prepare("SELECT * FROM plugin_settings WHERE plugin_slug = ? ORDER BY setting_key ASC");
            $stmt->execute([$viewPlugin['slug']]);
            $viewSettings = $stmt->fetchAll(PDO::FETCH_ASSOC);
        }
    } catch (Throwable $e) {}
}

$availableEffects = count(getEffectPlugins());

/* Preset background images (4K) */
$backgroundPresets = [
    'Christmas Snowy Forest'   => 'https://images.unsplash.com/photo-1543589077-47d81606c1bf?w=2560&q=90',
    'Christmas Tree Lights'    => 'https://images.unsplash.com/photo-1512389142860-9c449e58a543?w=2560&q=90',
    'Winter Night'             => 'https://images.unsplash.com/photo-1482330454287-3dcc23b3d10e?w=2560&q=90',
    'Christmas Gifts'          => 'https://images.unsplash.com/photo-1513297887119-d46091b24bfa?w=2560&q=90',
    'Fireplace Warmth'         => 'https://images.unsplash.com/photo-1512389142860-9c449e58a543?w=2560&q=90',
    'Snowy Village'            => 'https://images.unsplash.com/photo-1543589077-47d81606c1bf?w=2560&q=90',
    'Christmas Bokeh'          => 'https://images.unsplash.com/photo-1513297887119-d46091b24bfa?w=2560&q=90',
    'Candy Canes'              => 'https://images.unsplash.com/photo-1543589077-47d81606c1bf?w=2560&q=90',
];
?>
<!DOCTYPE html>
<html lang="en" class="dark">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Plugins Manager - WebEngine Admin</title>

<script src="https://cdn.tailwindcss.com"></script>
<link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.4.0/css/all.min.css">
<link rel="stylesheet" href="https://fonts.googleapis.com/css2?family=Inter:wght@300;400;500;600;700;800&display=swap">

<script>
    tailwind.config = {
        darkMode: 'class',
        theme: { extend: { fontFamily: { sans: ['Inter', 'sans-serif'] } } }
    }
</script>

<style>
    body {
        background-image: url('https://i.imgur.com/r6MVsqi.png');
        background-size: cover;
        background-position: center;
        background-attachment: fixed;
        font-family: 'Inter', sans-serif;
    }
    body::before {
        content: ""; position: fixed; inset: 0;
        background: rgba(0, 0, 0, .45); z-index: -1;
    }
    .glass-card {
        background: rgba(10, 15, 29, .55);
        backdrop-filter: blur(16px);
        border: 1px solid rgba(255, 255, 255, .10);
        box-shadow: 0 8px 32px rgba(0, 0, 0, .45);
    }
    .glass-hover { transition: .3s ease; }
    .glass-hover:hover {
        transform: translateY(-3px);
        border-color: rgba(99, 102, 241, .45);
        box-shadow: 0 0 25px rgba(99, 102, 241, .25);
    }
    ::-webkit-scrollbar { width: 6px; height: 6px; }
    ::-webkit-scrollbar-track { background: #050811; }
    ::-webkit-scrollbar-thumb { background: rgba(99, 102, 241, .45); border-radius: 10px; }
    #sidebar { transition: .3s ease; }

    @keyframes pulse-green {
        0%, 100% { box-shadow: 0 0 0 0 rgba(16, 185, 129, 0.7); }
        50%      { box-shadow: 0 0 0 10px rgba(16, 185, 129, 0); }
    }
    .pulse-green { animation: pulse-green 2s infinite; }

    .plg-modal-backdrop {
        display: none;
        position: fixed; inset: 0;
        background: rgba(0, 0, 0, .8);
        backdrop-filter: blur(8px);
        z-index: 9999;
        align-items: center; justify-content: center;
        padding: 20px;
    }
    .plg-modal-backdrop.active { display: flex; }
    .plg-modal {
        background: linear-gradient(135deg, rgba(15, 23, 42, .98), rgba(10, 15, 29, .98));
        border: 1px solid rgba(99, 102, 241, .3);
        border-radius: 20px;
        padding: 32px; max-width: 480px; width: 100%;
        text-align: center;
        box-shadow: 0 25px 80px rgba(0, 0, 0, .8);
    }
    .plg-modal-icon {
        width: 72px; height: 72px; margin: 0 auto 20px;
        border-radius: 20px;
        background: linear-gradient(135deg, #6366f1, #a855f7);
        display: flex; align-items: center; justify-content: center;
        font-size: 32px; color: #fff;
    }
    .plg-modal-title { font-size: 22px; font-weight: 900; color: #fff; margin: 0 0 12px; }
    .plg-modal-text { font-size: 15px; color: #94a3b8; line-height: 1.6; margin: 0 0 28px; }
    .plg-modal-actions { display: flex; gap: 12px; justify-content: center; }
    .plg-btn {
        flex: 1; padding: 14px 24px; border-radius: 12px;
        font-weight: 700; font-size: 14px; border: none;
        cursor: pointer;
        display: inline-flex; align-items: center; justify-content: center;
        gap: 8px; transition: all .2s ease; font-family: inherit;
    }
    .plg-btn-yes { background: linear-gradient(135deg, #10b981, #059669); color: #fff; }
    .plg-btn-yes:hover { transform: translateY(-2px); }
    .plg-btn-no { background: linear-gradient(135deg, #ef4444, #dc2626); color: #fff; }
    .plg-btn-no:hover { transform: translateY(-2px); }

    .plg-switch { position: relative; display: inline-block; width: 48px; height: 26px; }
    .plg-switch input { opacity: 0; width: 0; height: 0; }
    .plg-slider {
        position: absolute; cursor: pointer; inset: 0;
        background-color: #334155; transition: .3s; border-radius: 26px;
    }
    .plg-slider:before {
        position: absolute; content: ""; height: 20px; width: 20px;
        left: 3px; bottom: 3px; background-color: white;
        transition: .3s; border-radius: 50%;
    }
    .plg-switch input:checked + .plg-slider { background-color: #10b981; }
    .plg-switch input:checked + .plg-slider:before { transform: translateX(22px); }
    .plg-switch.loading { opacity: .6; pointer-events: none; }

    /* Background preview */
    .bg-preview {
        width: 100%;
        height: 180px;
        border-radius: 12px;
        background-size: cover;
        background-position: center;
        border: 2px solid rgba(99, 102, 241, .4);
        margin-bottom: 16px;
        position: relative;
        overflow: hidden;
    }
    .bg-preview::after {
        content: "LIVE PREVIEW";
        position: absolute;
        bottom: 8px; right: 8px;
        background: rgba(0, 0, 0, .7);
        color: #fff;
        font-size: 10px;
        padding: 4px 8px;
        border-radius: 6px;
        font-weight: 700;
        letter-spacing: 0.05em;
    }
    .bg-preset {
        width: 100%;
        height: 90px;
        border-radius: 8px;
        background-size: cover;
        background-position: center;
        border: 2px solid transparent;
        cursor: pointer;
        transition: all .2s ease;
        position: relative;
    }
    .bg-preset:hover {
        border-color: #6366f1;
        transform: scale(1.03);
    }
    .bg-preset.selected {
        border-color: #10b981;
        box-shadow: 0 0 0 3px rgba(16, 185, 129, .3);
    }
    .bg-preset-label {
        position: absolute;
        bottom: 0; left: 0; right: 0;
        background: linear-gradient(to top, rgba(0,0,0,.9), transparent);
        color: #fff;
        font-size: 10px;
        padding: 12px 6px 4px;
        text-align: center;
        font-weight: 700;
        border-radius: 0 0 8px 8px;
    }

    /* ===== Flash auto-hide ===== */
    #plgFlash {
        transition: opacity .5s ease, transform .5s ease,
                    max-height .5s ease, padding .5s ease,
                    margin .5s ease;
        max-height: 200px;
        overflow: hidden;
    }
    #plgFlash.plg-flash-hide {
        opacity: 0;
        transform: translateY(-10px);
        max-height: 0;
        padding-top: 0;
        padding-bottom: 0;
        margin-bottom: 0;
    }

    /* ===== Toast (for AJAX toggle) ===== */
    #plgToast {
        position: fixed;
        top: 24px;
        right: 24px;
        z-index: 10000;
        display: flex;
        flex-direction: column;
        gap: 10px;
        pointer-events: none;
    }
    .plg-toast-item {
        min-width: 280px;
        max-width: 420px;
        padding: 14px 18px;
        border-radius: 14px;
        background: rgba(10, 15, 29, .95);
        border: 1px solid rgba(255, 255, 255, .12);
        border-left: 4px solid #10b981;
        box-shadow: 0 12px 40px rgba(0, 0, 0, .6);
        backdrop-filter: blur(16px);
        color: #fff;
        font-size: 13px;
        font-weight: 500;
        display: flex;
        align-items: center;
        gap: 10px;
        transform: translateX(120%);
        opacity: 0;
        transition: transform .35s ease, opacity .35s ease;
        pointer-events: auto;
    }
    .plg-toast-item.show {
        transform: translateX(0);
        opacity: 1;
    }
    .plg-toast-item.hide {
        transform: translateX(120%);
        opacity: 0;
    }
    .plg-toast-item.error  { border-left-color: #ef4444; }
    .plg-toast-item.success{ border-left-color: #10b981; }
    .plg-toast-item.info   { border-left-color: #06b6d4; }
    .plg-toast-item i { font-size: 16px; }
    .plg-toast-item.success i { color: #34d399; }
    .plg-toast-item.error i   { color: #f87171; }
    .plg-toast-item.info i    { color: #22d3ee; }
</style>
</head>

<body class="text-slate-200 min-h-screen flex flex-col md:flex-row overflow-x-hidden font-sans">

<div id="sidebar-overlay" class="fixed inset-0 bg-black/80 backdrop-blur-md z-20 hidden md:hidden"></div>

<!-- TOAST CONTAINER -->
<div id="plgToast"></div>

<!-- SIDEBAR -->
<aside id="sidebar" class="fixed md:static inset-y-0 left-0 z-30 w-64 bg-[#070b14]/80 backdrop-blur-xl border-r border-slate-700/50 flex flex-col transform -translate-x-full md:translate-x-0 transition duration-300 shrink-0">
    <div class="flex flex-col h-full">
        <div class="h-20 flex items-center px-6 border-b border-slate-700/50 justify-between">
            <div class="flex items-center gap-3">
                <div class="bg-gradient-to-tr from-purple-600 via-indigo-600 to-cyan-500 p-2.5 rounded-xl shadow-lg">
                    <i class="fa-solid fa-cube text-xl text-white"></i>
                </div>
                <span class="font-extrabold text-xl text-white tracking-wide">
                    Web<span class="text-indigo-400">Engine</span>
                </span>
            </div>
            <button id="close-sidebar" class="md:hidden text-slate-400 hover:text-white">
                <i class="fa-solid fa-xmark text-xl"></i>
            </button>
        </div>

        <div class="flex-1 overflow-y-auto px-4 py-6 space-y-6">
            <div>
                <span class="text-[10px] uppercase tracking-widest text-slate-400 px-3 font-bold">Home</span>
                <a href="index.php" class="mt-2 flex items-center gap-3 px-4 py-3 rounded-xl text-slate-300 hover:bg-slate-800/60 hover:text-indigo-400 transition">
                    <i class="fa-solid fa-chart-pie w-5"></i><span>Dashboard</span>
                </a>
                <a href="users.php" class="flex items-center gap-3 px-4 py-3 rounded-xl text-slate-300 hover:bg-slate-800/60 hover:text-indigo-400 transition">
                    <i class="fa-solid fa-users w-5"></i><span>Users</span>
                </a>
                <a href="messages.php" class="flex items-center gap-3 px-4 py-3 rounded-xl text-slate-300 hover:bg-slate-800/60 transition">
                    <i class="fa-solid fa-comments w-5"></i><span>Messages</span>
                </a>
                <a href="chat.php" class="flex items-center gap-3 px-4 py-3 rounded-xl text-slate-300 hover:bg-slate-800/60 hover:text-indigo-400 transition">
                    <i class="fa-solid fa-comment-dots w-5"></i><span>Chat</span>
                </a>
                <a href="online.php" class="flex items-center gap-3 px-4 py-3 rounded-xl text-slate-300 hover:text-emerald-400 hover:bg-slate-800/60 transition">
                    <i class="fa-solid fa-circle text-emerald-400 text-xs"></i><span>Online Users</span>
                </a>
                <a href="products.php" class="flex items-center gap-3 px-4 py-3 rounded-xl text-slate-300 hover:bg-slate-800/60 hover:text-indigo-400 transition">
                    <i class="fa-solid fa-cart-shopping w-5"></i><span>Products</span>
                </a>
            </div>

            <div>
                <span class="text-[10px] uppercase tracking-widest text-slate-400 px-3 font-bold">Management</span>
                <a href="backup.php" class="mt-2 flex items-center gap-3 px-4 py-3 rounded-xl text-slate-300 hover:bg-slate-800/60 transition">
                    <i class="fa-solid fa-database w-5"></i><span>Backup</span>
                </a>
                <a href="restore.php" class="flex items-center gap-3 px-4 py-3 rounded-xl text-slate-300 hover:bg-slate-800/60 hover:text-emerald-400 transition">
                    <i class="fa-solid fa-rotate-left w-5"></i><span>Restore Website</span>
                </a>
                <a href="plugins.php" class="flex items-center gap-3 px-4 py-3 rounded-xl bg-gradient-to-r from-indigo-600 to-purple-600 text-white shadow-lg">
                    <i class="fa-solid fa-puzzle-piece w-5"></i><span>Plugins</span>
                </a>
                <a href="updates.php" class="flex items-center gap-3 px-4 py-3 rounded-xl text-slate-300 hover:bg-slate-800/60 hover:text-emerald-400 transition">
                    <i class="fa-solid fa-cloud-arrow-down w-5"></i><span>Update</span>
                </a>
                <a href="api.php" class="flex items-center gap-3 px-4 py-3 rounded-xl text-slate-300 hover:bg-slate-800/60 hover:text-cyan-400 transition">
                    <i class="fa-solid fa-plug w-5"></i><span>API</span>
                </a>
                <a href="seo.php" class="flex items-center gap-3 px-4 py-3 rounded-xl text-slate-300 hover:bg-slate-800/60 transition">
                    <i class="fa-solid fa-search w-5"></i><span>SEO Tools</span>
                </a>
            </div>

            <div>
                <span class="text-[10px] uppercase tracking-widest text-slate-400 px-3 font-bold">Security</span>
                <a href="security.php" class="mt-2 flex items-center gap-3 px-4 py-3 rounded-xl text-slate-300 hover:bg-slate-800/60 transition">
                    <i class="fa-solid fa-shield-halved w-5"></i><span>Security Scan</span>
                </a>
                <a href="security_center.php" class="flex items-center gap-3 px-4 py-3 rounded-xl text-slate-300 hover:bg-slate-800/60 hover:text-rose-400 transition">
                    <i class="fa-solid fa-lock w-5"></i><span>Security</span>
                </a>
                <a href="errors.php" class="flex items-center gap-3 px-4 py-3 rounded-xl text-slate-300 hover:bg-slate-800/60 hover:text-amber-400 transition">
                    <i class="fa-solid fa-triangle-exclamation w-5"></i><span>Errors</span>
                </a>
                <a href="virus_scan.php" class="flex items-center gap-3 px-4 py-3 rounded-xl text-slate-300 hover:bg-slate-800/60 hover:text-rose-400 transition">
                    <i class="fa-solid fa-virus w-5"></i><span>Virus Scan</span>
                </a>
                <a href="ip_address.php" class="flex items-center gap-3 px-4 py-3 rounded-xl text-slate-300 hover:bg-slate-800/60 hover:text-cyan-400 transition">
                    <i class="fa-solid fa-network-wired w-5"></i><span>IP Address</span>
                </a>
                <a href="logs.php" class="flex items-center gap-3 px-4 py-3 rounded-xl text-slate-300 hover:bg-slate-800/60 transition">
                    <i class="fa-solid fa-file-lines w-5"></i><span>Logs</span>
                </a>
            </div>

            <hr class="border-slate-700/50">

            <a href="settings.php" class="flex items-center gap-3 px-4 py-3 rounded-xl text-slate-300 hover:bg-slate-800/60 transition">
                <i class="fa-solid fa-gear w-5"></i><span>Settings</span>
            </a>
            <a href="logout.php" class="flex items-center gap-3 px-4 py-3 rounded-xl text-rose-400 hover:bg-rose-500/10 transition">
                <i class="fa-solid fa-right-from-bracket w-5"></i><span>Logout</span>
            </a>
        </div>
    </div>
</aside>

<!-- MAIN -->
<div class="flex-1 flex flex-col min-h-screen relative z-10">

    <header class="h-20 px-6 flex items-center justify-between bg-[#070b14]/50 border-b border-slate-700/50 backdrop-blur-xl sticky top-0 z-10">
        <div class="flex items-center gap-4">
            <button id="open-sidebar" class="md:hidden text-slate-300">
                <i class="fa-solid fa-bars text-xl"></i>
            </button>
            <div>
                <h1 class="text-xl font-extrabold text-white flex items-center gap-3">
                    <i class="fa-solid fa-puzzle-piece text-indigo-400"></i>
                    Plugins Manager
                </h1>
                <p class="text-xs text-slate-400">3D/4K effect plugins - auto-generate & manage</p>
            </div>
        </div>

        <div class="flex items-center gap-4">
            <a href="/"
               class="hidden sm:flex items-center gap-2 px-4 py-2.5 rounded-xl bg-gradient-to-r from-emerald-600 to-teal-600 hover:from-emerald-500 hover:to-teal-500 text-white text-sm font-semibold shadow-lg transition-all hover:scale-105">
                <i class="fa-solid fa-house"></i>
                <span>Back to Website</span>
            </a>

            <div class="flex items-center gap-3 border-l border-slate-700 pl-4">
                <img src="<?= $avatar ?>" class="w-11 h-11 rounded-xl object-cover ring-2 ring-indigo-500/50" alt="Avatar">
                <div class="hidden md:block">
                    <div class="text-sm font-bold text-white"><?= $username ?></div>
                    <div class="text-xs text-indigo-400 uppercase font-semibold">Super Admin</div>
                </div>
            </div>
        </div>
    </header>

    <main class="flex-1 p-6 space-y-6 max-w-7xl mx-auto w-full">

        <!-- FLASH (auto-hide after 2s) -->
        <?php if ($flash): ?>
            <div id="plgFlash"
                 class="glass-card rounded-2xl p-4 border-l-4
                <?= $flash['type'] === 'success' ? 'border-emerald-500' : ($flash['type'] === 'error' ? 'border-rose-500' : 'border-cyan-500') ?>">
                <div class="flex items-center gap-3">
                    <i class="fa-solid <?= $flash['type'] === 'success'
                        ? 'fa-circle-check text-emerald-400'
                        : ($flash['type'] === 'error' ? 'fa-circle-exclamation text-rose-400' : 'fa-circle-info text-cyan-400') ?> text-xl"></i>
                    <span class="text-sm font-medium text-white flex-1"><?= htmlspecialchars($flash['message'], ENT_QUOTES, 'UTF-8') ?></span>
                    <button type="button" onclick="hidePlgFlash()"
                            class="text-slate-400 hover:text-white transition shrink-0"
                            title="Close">
                        <i class="fa-solid fa-xmark"></i>
                    </button>
                </div>
            </div>

            <script>
                (function () {
                    var flash = document.getElementById('plgFlash');
                    if (!flash) return;

                    var autoTimer = setTimeout(function () {
                        hidePlgFlash();
                    }, 2000);

                    window.hidePlgFlash = function () {
                        clearTimeout(autoTimer);
                        if (!flash || flash.classList.contains('plg-flash-hide')) return;
                        flash.classList.add('plg-flash-hide');

                        setTimeout(function () {
                            if (flash && flash.parentNode) {
                                flash.parentNode.removeChild(flash);
                            }
                        }, 500);
                    };
                })();
            </script>
        <?php endif; ?>

        <!-- EFFECT GENERATOR PANEL -->
        <div class="glass-card rounded-3xl p-6 border-l-4 border-pink-500">
            <div class="flex flex-wrap items-center justify-between gap-4">
                <div class="flex items-center gap-4">
                    <div class="p-3 rounded-xl bg-gradient-to-br from-pink-500 to-rose-500 shadow-lg">
                        <i class="fa-solid fa-wand-magic-sparkles text-white text-2xl"></i>
                    </div>
                    <div>
                        <h2 class="text-xl font-black text-white">Auto-Generate Effect Plugins (3D/4K)</h2>
                        <p class="text-xs text-slate-400">
                            <?= $availableEffects ?> effects - Rain, Snow, Santa, Tree, Witch, C4, and more
                        </p>
                    </div>
                </div>

                <div class="flex flex-wrap items-center gap-2">
                    <form method="post" class="inline" data-confirm="Generate ALL <?= $availableEffects ?> effect plugins?">
                        <input type="hidden" name="action" value="generate_all_effects">
                        <button type="submit"
                                class="px-6 py-3 rounded-xl bg-gradient-to-r from-pink-600 to-rose-600 hover:from-pink-500 hover:to-rose-500 text-white font-bold text-sm shadow-lg transition-all hover:scale-105">
                            <i class="fa-solid fa-wand-magic-sparkles mr-2"></i>
                            Generate All <?= $availableEffects ?> Effects
                        </button>
                    </form>

                    <?php if ($statEffects > 0): ?>
                        <form method="post" class="inline" data-confirm="Delete ALL effect plugins?">
                            <input type="hidden" name="action" value="delete_all_effects">
                            <button type="submit"
                                    class="px-4 py-3 rounded-xl bg-slate-800 hover:bg-slate-700 text-rose-300 font-bold text-sm transition">
                                <i class="fa-solid fa-trash mr-1"></i>Delete All
                            </button>
                        </form>
                    <?php endif; ?>
                </div>
            </div>

            <div class="mt-4 p-3 rounded-xl bg-slate-900/50 border border-slate-700/50">
                <p class="text-xs text-slate-400">
                    <i class="fa-solid fa-info-circle text-cyan-400 mr-1"></i>
                    All effects now support <strong class="text-white">3D/4K image URLs</strong> (PNG with transparency, GIF, WEBP).
                    Configure each plugin to change its image, size, speed, and position.
                </p>
            </div>
        </div>

        <!-- STATS -->
        <div class="grid grid-cols-2 sm:grid-cols-4 gap-4">
            <div class="glass-card glass-hover rounded-2xl p-5">
                <div class="flex justify-between">
                    <span class="text-xs text-slate-400 font-bold">TOTAL</span>
                    <i class="fa-solid fa-puzzle-piece text-slate-400"></i>
                </div>
                <h2 class="text-3xl font-black text-white mt-3" id="statTotal"><?= $statTotal ?></h2>
            </div>
            <div class="glass-card glass-hover rounded-2xl p-5">
                <div class="flex justify-between">
                    <span class="text-xs text-emerald-400 font-bold">ACTIVE</span>
                    <i class="fa-solid fa-circle-check text-emerald-400"></i>
                </div>
                <h2 class="text-3xl font-black text-white mt-3" id="statActive"><?= $statActive ?></h2>
            </div>
            <div class="glass-card glass-hover rounded-2xl p-5">
                <div class="flex justify-between">
                    <span class="text-xs text-yellow-400 font-bold">INACTIVE</span>
                    <i class="fa-solid fa-circle-pause text-yellow-400"></i>
                </div>
                <h2 class="text-3xl font-black text-white mt-3" id="statInactive"><?= $statInactive ?></h2>
            </div>
            <div class="glass-card glass-hover rounded-2xl p-5">
                <div class="flex justify-between">
                    <span class="text-xs text-pink-400 font-bold">EFFECTS</span>
                    <i class="fa-solid fa-wand-magic-sparkles text-pink-400"></i>
                </div>
                <h2 class="text-3xl font-black text-white mt-3"><?= $statEffects ?></h2>
            </div>
        </div>

        <!-- VIEW SINGLE PLUGIN SETTINGS -->
        <?php if ($viewPlugin): ?>
            <div class="glass-card rounded-3xl p-6 border-l-4 border-indigo-500">
                <div class="flex items-start justify-between gap-4 mb-4">
                    <div class="flex items-center gap-4">
                        <div class="w-16 h-16 rounded-2xl bg-gradient-to-br from-indigo-600 to-purple-600 flex items-center justify-center">
                            <i class="fa-solid <?= htmlspecialchars($viewPlugin['icon']) ?> text-white text-2xl"></i>
                        </div>
                        <div>
                            <h2 class="text-2xl font-black text-white"><?= htmlspecialchars($viewPlugin['name']) ?></h2>
                            <p class="text-sm text-slate-400">
                                slug: <span class="font-mono text-cyan-400"><?= htmlspecialchars($viewPlugin['slug']) ?></span>
                                - v<?= htmlspecialchars($viewPlugin['version']) ?>
                            </p>
                        </div>
                    </div>
                    <a href="plugins.php" class="p-2 rounded-lg text-slate-400 hover:text-white hover:bg-slate-800 transition">
                        <i class="fa-solid fa-xmark"></i>
                    </a>
                </div>

                <div class="pt-4 border-t border-slate-700/50">
                    <h3 class="text-white font-bold mb-3 flex items-center gap-2">
                        <i class="fa-solid fa-gear text-cyan-400"></i>
                        Plugin Settings
                        <span class="text-xs text-slate-500 font-normal">(<?= count($viewSettings) ?>)</span>
                    </h3>

                    <?php
                    /* Get current image URL for preview */
                    $currentImageUrl = '';
                    $currentBgUrl = '';
                    foreach ($viewSettings as $s) {
                        if ($s['setting_key'] === 'image_url') $currentImageUrl = $s['setting_value'];
                        if ($s['setting_key'] === 'image_urls' && $currentImageUrl === '') $currentImageUrl = explode(',', $s['setting_value'])[0] ?? '';
                    }

                    /* Background preview for effect-christmas-bg */
                    if ($viewPlugin['slug'] === 'effect-christmas-bg' && $currentImageUrl !== ''):
                    ?>
                        <div class="bg-preview" id="bgPreview" style="background-image: url('<?= htmlspecialchars($currentImageUrl, ENT_QUOTES) ?>');"></div>
                    <?php elseif ($viewPlugin['slug'] === 'effect-christmas-bg'): ?>
                        <div class="bg-preview" id="bgPreview"></div>
                    <?php endif; ?>

                    <!-- Background Presets -->
                    <?php if ($viewPlugin['slug'] === 'effect-christmas-bg'): ?>
                        <div class="mb-5">
                            <label class="block text-xs font-semibold text-slate-300 mb-2 uppercase">
                                <i class="fa-solid fa-images text-pink-400"></i>
                                Quick Background Presets (4K)
                            </label>
                            <div class="grid grid-cols-2 md:grid-cols-4 gap-2">
                                <?php foreach ($backgroundPresets as $label => $url): ?>
                                    <div class="bg-preset"
                                         style="background-image: url('<?= htmlspecialchars($url, ENT_QUOTES) ?>');"
                                         data-url="<?= htmlspecialchars($url, ENT_QUOTES) ?>"
                                         onclick="selectBgPreset(this)">
                                        <div class="bg-preset-label"><?= htmlspecialchars($label) ?></div>
                                    </div>
                                <?php endforeach; ?>
                            </div>
                        </div>
                    <?php endif; ?>

                    <?php if (empty($viewSettings)): ?>
                        <div class="p-4 rounded-xl bg-slate-900/40 border border-slate-700/50 text-center text-sm text-slate-500">
                            This plugin has no settings.
                        </div>
                    <?php else: ?>
                        <form method="post" class="space-y-3" id="settingsForm">
                            <input type="hidden" name="action" value="save_settings">
                            <input type="hidden" name="slug" value="<?= htmlspecialchars($viewPlugin['slug']) ?>">

                            <?php foreach ($viewSettings as $s):
                                $key = $s['setting_key'];
                                $val = (string)$s['setting_value'];
                                $isImageUrl = (str_contains($key, 'image') || str_contains($key, 'url'));
                            ?>
                                <div>
                                    <label class="block text-xs font-semibold text-slate-300 mb-1 uppercase font-mono">
                                        <?= htmlspecialchars($key) ?>
                                        <?php if (str_contains($key, 'image')): ?>
                                            <span class="text-pink-400 normal-case font-normal text-[10px]">- supports 3D/4K PNG, GIF, WEBP</span>
                                        <?php endif; ?>
                                    </label>
                                    <?php if ($isImageUrl && strlen($val) > 80): ?>
                                        <textarea name="settings[<?= htmlspecialchars($key) ?>]"
                                                  rows="2"
                                                  class="w-full px-4 py-3 rounded-xl bg-slate-900/60 border border-slate-700 focus:border-indigo-500 outline-none text-white text-sm font-mono"
                                                  oninput="updatePreview('<?= htmlspecialchars($key) ?>', this.value)"><?= htmlspecialchars($val) ?></textarea>
                                    <?php else: ?>
                                        <input type="text" name="settings[<?= htmlspecialchars($key) ?>]"
                                               value="<?= htmlspecialchars($val) ?>"
                                               class="w-full px-4 py-3 rounded-xl bg-slate-900/60 border border-slate-700 focus:border-indigo-500 outline-none text-white text-sm <?= $isImageUrl ? 'font-mono' : '' ?>"
                                               oninput="updatePreview('<?= htmlspecialchars($key) ?>', this.value)">
                                    <?php endif; ?>
                                </div>
                            <?php endforeach; ?>

                            <button type="submit"
                                    class="px-6 py-3 rounded-xl bg-gradient-to-r from-indigo-600 to-purple-600 hover:from-indigo-500 hover:to-purple-500 text-white font-bold text-sm shadow-lg transition">
                                <i class="fa-solid fa-floppy-disk mr-2"></i>Save Settings
                            </button>
                        </form>
                    <?php endif; ?>
                </div>
            </div>
        <?php endif; ?>

        <!-- FILTERS -->
        <div class="glass-card rounded-3xl p-5">
            <form method="get" class="flex flex-wrap items-center gap-3">
                <div class="flex-1 min-w-[200px] relative">
                    <i class="fa-solid fa-magnifying-glass absolute left-4 top-1/2 -translate-y-1/2 text-slate-500"></i>
                    <input type="text" name="search"
                           value="<?= htmlspecialchars($search) ?>"
                           placeholder="Search plugins..."
                           class="w-full pl-11 pr-4 py-3 rounded-xl bg-slate-900/60 border border-slate-700 focus:border-indigo-500 outline-none text-white text-sm">
                </div>

                <select name="status" class="px-4 py-3 rounded-xl bg-slate-900/60 border border-slate-700 outline-none text-white text-sm">
                    <option value="">All Statuses</option>
                    <option value="active"   <?= $filterStatus === 'active' ? 'selected' : '' ?>>Active</option>
                    <option value="inactive" <?= $filterStatus === 'inactive' ? 'selected' : '' ?>>Inactive</option>
                </select>

                <button type="submit" class="px-6 py-3 rounded-xl bg-gradient-to-r from-indigo-600 to-purple-600 hover:from-indigo-500 hover:to-purple-500 text-white font-bold text-sm shadow-lg transition">
                    <i class="fa-solid fa-filter mr-2"></i>Filter
                </button>

                <a href="plugins.php" class="px-4 py-3 rounded-xl bg-slate-800 hover:bg-slate-700 text-slate-300 font-bold text-sm transition">
                    <i class="fa-solid fa-rotate-left"></i>
                </a>
            </form>
        </div>

        <!-- PLUGINS LIST -->
        <div class="glass-card rounded-3xl p-6">
            <div class="flex items-center justify-between mb-5">
                <h2 class="text-white font-bold flex items-center gap-2">
                    <i class="fa-solid fa-list text-indigo-400"></i>
                    Installed Plugins
                    <span class="text-xs text-slate-400 font-normal">(<?= count($plugins) ?>)</span>
                </h2>
            </div>

            <?php if (empty($plugins)): ?>
                <div class="text-center py-16 text-slate-500">
                    <i class="fa-solid fa-puzzle-piece text-6xl mb-4 text-indigo-500/30"></i>
                    <h3 class="text-white font-bold text-xl mb-2">No plugins installed</h3>
                    <p class="text-sm">Click the <strong class="text-pink-400">"Generate All Effects"</strong> button above.</p>
                </div>
            <?php else: ?>
                <div class="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-4">
                    <?php foreach ($plugins as $p):
                        $isActive = $p['status'] === 'active';
                        $isRequired = (int)$p['is_required'] === 1;
                        $isEffect = str_starts_with($p['slug'], 'effect-');
                    ?>
                        <div class="rounded-2xl bg-slate-900/40 border <?= $isActive ? 'border-emerald-500/40' : 'border-slate-700/50' ?> p-5 relative transition hover:border-indigo-500/40"
                             data-plugin-card="<?= (int)$p['id'] ?>">

                            <span class="plg-active-dot absolute top-3 right-3 w-3 h-3 bg-emerald-500 rounded-full pulse-green <?= $isActive ? '' : 'hidden' ?>"></span>

                            <div class="flex items-start gap-3 mb-3">
                                <div class="w-12 h-12 rounded-xl <?= $isEffect ? 'bg-gradient-to-br from-pink-600 to-rose-600' : ($isActive ? 'bg-gradient-to-br from-emerald-600 to-teal-600' : 'bg-gradient-to-br from-slate-700 to-slate-800') ?> flex items-center justify-center shrink-0">
                                    <i class="fa-solid <?= htmlspecialchars($p['icon']) ?> text-white text-lg"></i>
                                </div>
                                <div class="flex-1 min-w-0">
                                    <div class="text-sm font-bold text-white truncate"><?= htmlspecialchars($p['name']) ?></div>
                                    <div class="text-[10px] text-slate-500 font-mono truncate"><?= htmlspecialchars($p['slug']) ?></div>
                                </div>
                            </div>

                            <div class="text-xs text-slate-400 mb-3 min-h-[36px]">
                                <?= htmlspecialchars($p['description'] ?? '') ?>
                            </div>

                            <div class="flex items-center gap-2 flex-wrap mb-3 text-[10px]">
                                <span class="px-2 py-0.5 rounded bg-slate-700/50 text-slate-300 font-mono">
                                    v<?= htmlspecialchars($p['version']) ?>
                                </span>
                                <span class="<?= $isEffect ? 'bg-pink-500/15 text-pink-300' : 'bg-cyan-500/15 text-cyan-300' ?> px-2 py-0.5 rounded font-bold uppercase">
                                    <?= htmlspecialchars($p['category']) ?>
                                </span>
                            </div>

                            <div class="flex items-center gap-2 pt-3 border-t border-slate-700/50">
                                <!-- AJAX TOGGLE (no page reload) -->
                                <label class="plg-switch" data-toggle-id="<?= (int)$p['id'] ?>">
                                    <input type="checkbox" <?= $isActive ? 'checked' : '' ?>
                                           onchange="ajaxToggle(this, <?= (int)$p['id'] ?>)">
                                    <span class="plg-slider"></span>
                                </label>

                                <a href="?view=<?= (int)$p['id'] ?>"
                                   class="p-2 rounded-lg text-slate-400 hover:text-indigo-400 hover:bg-indigo-500/10 transition"
                                   title="Configure">
                                    <i class="fa-solid fa-gear text-xs"></i>
                                </a>

                                <form method="post" class="inline ml-auto" data-confirm="Delete plugin?">
                                    <input type="hidden" name="action" value="delete">
                                    <input type="hidden" name="id" value="<?= (int)$p['id'] ?>">
                                    <input type="hidden" name="remove_files" value="1">
                                    <button type="submit" class="p-2 rounded-lg text-slate-400 hover:text-rose-400 hover:bg-rose-500/10 transition">
                                        <i class="fa-solid fa-trash text-xs"></i>
                                    </button>
                                </form>
                            </div>
                        </div>
                    <?php endforeach; ?>
                </div>
            <?php endif; ?>
        </div>

        <!-- INFO -->
        <div class="glass-card rounded-3xl p-6 border-l-4 border-pink-500">
            <div class="flex items-start gap-4">
                <div class="p-3 rounded-xl bg-pink-500/20 shrink-0">
                    <i class="fa-solid fa-image text-pink-400 text-xl"></i>
                </div>
                <div class="flex-1 text-sm text-slate-300">
                    <h3 class="text-white font-bold mb-2">3D / 4K Images - Where to find them</h3>
                    <ul class="space-y-2 text-[12px]">
                        <li>
                            <i class="fa-solid fa-link text-cyan-400 mr-2"></i>
                            <strong class="text-white">Free PNG icons:</strong>
                            <a href="https://www.flaticon.com/" target="_blank" class="text-cyan-400 hover:underline">flaticon.com</a> -
                            <a href="https://icons8.com/" target="_blank" class="text-cyan-400 hover:underline">icons8.com</a> -
                            <a href="https://www.freepik.com/" target="_blank" class="text-cyan-400 hover:underline">freepik.com</a>
                        </li>
                        <li>
                            <i class="fa-solid fa-link text-pink-400 mr-2"></i>
                            <strong class="text-white">4K backgrounds:</strong>
                            <a href="https://unsplash.com/" target="_blank" class="text-pink-400 hover:underline">unsplash.com</a> -
                            <a href="https://www.pexels.com/" target="_blank" class="text-pink-400 hover:underline">pexels.com</a>
                        </li>
                        <li>
                            <i class="fa-solid fa-link text-yellow-400 mr-2"></i>
                            <strong class="text-white">Animated GIFs:</strong>
                            <a href="https://giphy.com/" target="_blank" class="text-yellow-400 hover:underline">giphy.com</a> -
                            <a href="https://tenor.com/" target="_blank" class="text-yellow-400 hover:underline">tenor.com</a>
                        </li>
                    </ul>
                    <p class="mt-3 text-[11px] text-slate-500">
                        <i class="fa-solid fa-info-circle mr-1"></i>
                        All image URLs must be direct links (ending in .png, .jpg, .gif, .webp).
                        Recommended for best quality: PNG with transparency, 512Ã—512 or 1024Ã—1024.
                    </p>
                </div>
            </div>
        </div>

    </main>
</div>

<!-- CONFIRM MODAL -->
<div id="plgConfirmModal" class="plg-modal-backdrop">
    <div class="plg-modal">
        <div class="plg-modal-icon">
            <i class="fa-solid fa-circle-question"></i>
        </div>
        <h2 class="plg-modal-title">Are you sure?</h2>
        <p class="plg-modal-text" id="plgModalText">Do you really want to do this action?</p>
        <div class="plg-modal-actions">
            <button type="button" id="plgModalYes" class="plg-btn plg-btn-yes">
                <i class="fa-solid fa-check"></i> Yes
            </button>
            <button type="button" id="plgModalNo" class="plg-btn plg-btn-no">
                <i class="fa-solid fa-xmark"></i> No
            </button>
        </div>
    </div>
</div>

<script>
    /* ==========================================================
       SIDEBAR
       ========================================================== */
    const sidebar = document.getElementById('sidebar');
    const overlay = document.getElementById('sidebar-overlay');
    const open    = document.getElementById('open-sidebar');
    const close   = document.getElementById('close-sidebar');

    function toggleSidebar() {
        sidebar.classList.toggle('-translate-x-full');
        overlay.classList.toggle('hidden');
    }
    open?.addEventListener('click', toggleSidebar);
    close?.addEventListener('click', toggleSidebar);
    overlay?.addEventListener('click', toggleSidebar);

    /* ==========================================================
       CONFIRM MODAL
       ========================================================== */
    (function() {
        'use strict';

        var modal     = document.getElementById('plgConfirmModal');
        var modalText = document.getElementById('plgModalText');
        var btnYes    = document.getElementById('plgModalYes');
        var btnNo     = document.getElementById('plgModalNo');

        if (!modal) return;

        var pendingForm = null;

        function openConfirm(form, message) {
            pendingForm = form;
            modalText.textContent = message || 'Do you really want to do this action?';
            modal.classList.add('active');
        }

        function closeConfirm() {
            modal.classList.remove('active');
            pendingForm = null;
        }

        btnYes?.addEventListener('click', function() {
            if (pendingForm) {
                pendingForm.dataset.confirmed = '1';
                pendingForm.submit();
            }
            closeConfirm();
        });

        btnNo?.addEventListener('click', closeConfirm);

        modal.addEventListener('click', function(e) {
            if (e.target === modal) closeConfirm();
        });

        document.addEventListener('keydown', function(e) {
            if (e.key === 'Escape') closeConfirm();
        });

        document.addEventListener('submit', function(e) {
            var form = e.target;
            if (!form.dataset.confirm) return;
            if (form.dataset.confirmed === '1') {
                delete form.dataset.confirmed;
                return;
            }
            e.preventDefault();
            openConfirm(form, form.dataset.confirm);
        }, true);
    })();

    /* ==========================================================
       AJAX TOGGLE (no page reload, no jump to top)
       ========================================================== */
    function ajaxToggle(checkbox, pluginId) {
        var label   = checkbox.closest('.plg-switch');
        var card    = document.querySelector('[data-plugin-card="' + pluginId + '"]');
        var dot     = card ? card.querySelector('.plg-active-dot') : null;
        var prevVal = !checkbox.checked;

        if (label) label.classList.add('loading');

        var formData = new FormData();
        formData.append('action', 'ajax_toggle');
        formData.append('id', pluginId);

        fetch(window.location.pathname + window.location.search, {
            method: 'POST',
            body: formData,
            credentials: 'same-origin',
            headers: { 'X-Requested-With': 'XMLHttpRequest' }
        })
        .then(function(r) { return r.json(); })
        .then(function(data) {
            if (label) label.classList.remove('loading');

            if (!data.ok) {
                /* revert checkbox on error */
                checkbox.checked = prevVal;
                showToast(data.message || 'Error', 'error');
                return;
            }

            var isActive = data.status === 'active';

            /* Update active dot */
            if (dot) {
                if (isActive) dot.classList.remove('hidden');
                else          dot.classList.add('hidden');
            }

            /* Update card border */
            if (card) {
                card.classList.toggle('border-emerald-500/40', isActive);
                card.classList.toggle('border-slate-700/50',   !isActive);
            }

            /* Update stats counters */
            updateStats(isActive);

            /* Toast notification (auto-hide after 2s) */
            showToast(data.message || 'Done', 'success');
        })
        .catch(function(err) {
            if (label) label.classList.remove('loading');
            checkbox.checked = prevVal;
            showToast('Network error: ' + err.message, 'error');
        });
    }

    /* Update active/inactive counters without page reload */
    function updateStats(isActive) {
        var actEl  = document.getElementById('statActive');
        var inaEl  = document.getElementById('statInactive');
        if (!actEl || !inaEl) return;

        var a = parseInt(actEl.textContent, 10) || 0;
        var i = parseInt(inaEl.textContent, 10) || 0;

        if (isActive) { a++; if (i > 0) i--; }
        else          { if (a > 0) a--; i++; }

        actEl.textContent = a;
        inaEl.textContent = i;
    }

    /* ==========================================================
       TOAST (auto-hide after 2s, animated)
       ========================================================== */
    function showToast(message, type) {
        type = type || 'success';
        var container = document.getElementById('plgToast');
        if (!container) return;

        var icons = {
            success: 'fa-circle-check',
            error:   'fa-circle-exclamation',
            info:    'fa-circle-info'
        };

        var toast = document.createElement('div');
        toast.className = 'plg-toast-item ' + type;
        toast.innerHTML = '<i class="fa-solid ' + (icons[type] || icons.success) + '"></i>' +
                          '<span></span>';
        toast.querySelector('span').textContent = message;

        container.appendChild(toast);

        /* Slide in */
        requestAnimationFrame(function() {
            toast.classList.add('show');
        });

        /* Hide after 2s */
        var hideTimer = setTimeout(function() {
            toast.classList.remove('show');
            toast.classList.add('hide');
            setTimeout(function() {
                if (toast.parentNode) toast.parentNode.removeChild(toast);
            }, 400);
        }, 2000);

        /* Manual close on click */
        toast.addEventListener('click', function() {
            clearTimeout(hideTimer);
            toast.classList.remove('show');
            toast.classList.add('hide');
            setTimeout(function() {
                if (toast.parentNode) toast.parentNode.removeChild(toast);
            }, 400);
        });
    }

    /* ==========================================================
       BACKGROUND PRESET SELECTION
       ========================================================== */
    function selectBgPreset(el) {
        var url = el.dataset.url;
        var input = document.querySelector('input[name="settings[image_url]"]')
                || document.querySelector('textarea[name="settings[image_url]"]');

        if (input) {
            input.value = url;
            input.dispatchEvent(new Event('input', { bubbles: true }));
        }

        var preview = document.getElementById('bgPreview');
        if (preview) {
            preview.style.backgroundImage = 'url("' + url + '")';
        }

        document.querySelectorAll('.bg-preset').forEach(function(p) {
            p.classList.remove('selected');
        });
        el.classList.add('selected');
    }

    /* ==========================================================
       LIVE PREVIEW FOR BACKGROUND
       ========================================================== */
    function updatePreview(key, value) {
        if (key !== 'image_url') return;
        var preview = document.getElementById('bgPreview');
        if (!preview) return;
        if (value && value.trim() !== '') {
            preview.style.backgroundImage = 'url("' + value.trim() + '")';
        } else {
            preview.style.backgroundImage = '';
        }
    }
</script>

</body>
</html>
