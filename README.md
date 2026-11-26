========== INDEX.HTML ==========
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <meta name="theme-color" content="#101820">
    <title>NovaHost Control Panel</title>
    <link rel="stylesheet" href="/style.css">
</head>
<body>
<div class="app-shell">

    <aside class="sidebar sidebar-left" aria-label="Workspace navigation">
        <div class="sidebar-brand">
            <div class="sidebar-brand-icon">
                <i class="fa-solid fa-server"></i>
            </div>
            <div class="sidebar-brand-text">
                <strong>NovaHost</strong>
                <span>CONTROL CENTER</span>
            </div>
        </div>

        <div class="sidebar-group">
            <div class="sidebar-heading">WORKSPACE</div>

            <button class="sidebar-link active" data-target="dashboard-top">
                <i class="fa-solid fa-gauge-high"></i>
                <span>Dashboard</span>
            </button>

            <button class="sidebar-link" data-target="module-section">
                <i class="fa-solid fa-server"></i>
                <span>Server Modules</span>
            </button>

            <button class="sidebar-link" data-tool="Projects">
                <i class="fa-solid fa-folder-open"></i>
                <span>Projects</span>
            </button>

            <button class="sidebar-link" data-tool="Databases">
                <i class="fa-solid fa-database"></i>
                <span>Databases</span>
            </button>

            <button class="sidebar-link" data-tool="Runtimes">
                <i class="fa-solid fa-cubes"></i>
                <span>Runtimes</span>
            </button>
        </div>

        <div class="sidebar-group">
            <div class="sidebar-heading">TOOLS</div>

            <button class="sidebar-link" data-tool="PHP Manager">
                <i class="fa-brands fa-php"></i>
                <span>PHP Manager</span>
            </button>

            <button class="sidebar-link" data-tool="SSL Manager">
                <i class="fa-solid fa-shield-halved"></i>
                <span>SSL Manager</span>
            </button>

            <button class="sidebar-link" data-tool="Backups">
                <i class="fa-solid fa-box-archive"></i>
                <span>Backups</span>
            </button>

            <button class="sidebar-link" data-tool="Settings">
                <i class="fa-solid fa-gear"></i>
                <span>Settings</span>
            </button>
        </div>

        <div class="sidebar-status">
            <span class="sidebar-status-dot"></span>
            <span>LOCAL ENVIRONMENT</span>
        </div>
    </aside>

    <div class="window">
    <header id="dashboard-top" class="topbar">
        <div class="brand">
            <i class="fa-solid fa-server"></i>
            <strong>NovaHost</strong>
            <span>Control Panel</span>
        </div>
        <div class="top-info">
            <span id="engine-status" class="engine-status">ENGINE NOT CONNECTED</span>
            <span id="clock">--:--:--</span>
        </div>
    </header>

    <main>
        <section id="module-section" class="module-section">
            <div class="section-title">
                <h2>Modules</h2>
                <span>Local development environment</span>
            </div>

            <div class="table-wrap">
                <table class="module-table">
                    <thead>
                        <tr>
                            <th>Module</th>
                            <th>PID(s)</th>
                            <th>Port(s)</th>
                            <th>Actions</th>
                        </tr>
                    </thead>
                    <tbody>
                        <tr>
                            <td><i class="fa-solid fa-globe module-icon blue"></i><strong>NovaHost Web Engine</strong></td>
                            <td class="mono" id="pid-engine">--</td>
                            <td class="mono">8080</td>
                            <td class="actions">
                                <button class="btn start" disabled>Start</button>
                                <button class="btn admin" disabled>Admin</button>
                                <button class="btn config" disabled>Config</button>
                                <button class="btn logs" disabled>Logs</button>
                            </td>
                        </tr>
                        <tr>
                            <td><i class="fa-brands fa-php module-icon purple"></i><strong>PHP Runtime</strong></td>
                            <td class="mono" id="pid-php">--</td>
                            <td class="mono">9075</td>
                            <td class="actions">
                                <button class="btn start" disabled>Start</button>
                                <button class="btn admin" disabled>Admin</button>
                                <button class="btn config" disabled>Config</button>
                                <button class="btn logs" disabled>Logs</button>
                            </td>
                        </tr>
                        <tr>
                            <td><i class="fa-solid fa-database module-icon orange"></i><strong>MariaDB</strong></td>
                            <td class="mono" id="pid-mariadb">--</td>
                            <td class="mono">3306</td>
                            <td class="actions">
                                <button class="btn start" disabled>Start</button>
                                <button class="btn admin" disabled>Admin</button>
                                <button class="btn config" disabled>Config</button>
                                <button class="btn logs" disabled>Logs</button>
                            </td>
                        </tr>
                        <tr>
                            <td><i class="fa-solid fa-layer-group module-icon red"></i><strong>Redis</strong></td>
                            <td class="mono" id="pid-redis">--</td>
                            <td class="mono">6379</td>
                            <td class="actions">
                                <button class="btn start" disabled>Start</button>
                                <button class="btn admin" disabled>Admin</button>
                                <button class="btn config" disabled>Config</button>
                                <button class="btn logs" disabled>Logs</button>
                            </td>
                        </tr>
                        <tr>
                            <td><i class="fa-brands fa-node-js module-icon green"></i><strong>Node.js</strong></td>
                            <td class="mono" id="pid-node">--</td>
                            <td class="mono">Dynamic</td>
                            <td class="actions">
                                <button class="btn start" disabled>Start</button>
                                <button class="btn admin" disabled>Admin</button>
                                <button class="btn config" disabled>Config</button>
                                <button class="btn logs" disabled>Logs</button>
                            </td>
                        </tr>
                        <tr>
                            <td><i class="fa-brands fa-python module-icon yellow"></i><strong>Python</strong></td>
                            <td class="mono" id="pid-python">--</td>
                            <td class="mono">Dynamic</td>
                            <td class="actions">
                                <button class="btn start" disabled>Start</button>
                                <button class="btn admin" disabled>Admin</button>
                                <button class="btn config" disabled>Config</button>
                                <button class="btn logs" disabled>Logs</button>
                            </td>
                        </tr>
                    </tbody>
                </table>
            </div>
            <div class="table-note">
                <i class="fa-solid fa-circle-info"></i>
                Process controls will activate after the NovaHost Process Manager is connected.
            </div>
        </section>

        <section class="tools-section">
            <h2>Tools</h2>
            <div class="tools-grid">
                <button class="tool-button" data-tool="Config"><i class="fa-solid fa-gear"></i>Config</button>
                <button class="tool-button" data-tool="Netstat"><i class="fa-solid fa-network-wired"></i>Netstat</button>
                <button class="tool-button" data-tool="Shell"><i class="fa-solid fa-terminal"></i>Shell</button>
                <button class="tool-button" data-tool="Explorer"><i class="fa-solid fa-folder-open"></i>Explorer</button>
                <button class="tool-button" data-tool="Projects"><i class="fa-solid fa-code"></i>Projects</button>
                <button class="tool-button" data-tool="Databases"><i class="fa-solid fa-database"></i>Databases</button>
                <button class="tool-button" data-tool="Runtimes"><i class="fa-solid fa-cubes"></i>Runtimes</button>
                <button class="tool-button" data-tool="PHP Manager"><i class="fa-brands fa-php"></i>PHP Manager</button>
                <button class="tool-button" data-tool="SSL Manager"><i class="fa-solid fa-shield-halved"></i>SSL Manager</button>
                <button class="tool-button" data-tool="Backups"><i class="fa-solid fa-box-archive"></i>Backups</button>
                <button class="tool-button" data-tool="Services"><i class="fa-solid fa-sliders"></i>Services</button>
                <button class="tool-button" data-tool="Help"><i class="fa-solid fa-circle-question"></i>Help</button>
            </div>
            <div id="tool-message" class="tool-message">Select a tool to open its workspace. Tools will be connected as their modules are implemented.</div>
        </section>

        <section class="logs-section">
            <div class="logs-header">
                <h2>NovaHost Log Console</h2>
                <div class="log-controls">
                    <button id="pause-logs" title="Pause logs"><i class="fa-solid fa-pause"></i></button>
                    <button id="clear-logs" title="Clear logs"><i class="fa-solid fa-trash"></i></button>
                    <button id="export-logs" title="Export logs"><i class="fa-solid fa-download"></i></button>
                </div>
            </div>
            <div class="log-toolbar">
                <div class="filters">
                    <button class="filter active" data-filter="all">All</button>
                    <button class="filter" data-filter="success">Start</button>
                    <button class="filter" data-filter="stop">Stop</button>
                    <button class="filter" data-filter="error">Errors</button>
                    <button class="filter" data-filter="info">Info</button>
                </div>
                <span id="log-count">0 entries</span>
            </div>
            <div id="log-list" class="log-list">
                <div class="log-empty">NovaHost is waiting for process events...</div>
            </div>
        </section>
    </main>

    <footer>
        <span>NovaHost Desktop Edition Â· v0.1.0</span>
        <span>Web root: D:\NovaHost\home\public</span>
        <span><b id="timezone-label">--</b> Â· <b id="footer-clock">--:--:--</b></span>
    </footer>
</div>

    <aside class="sidebar sidebar-right" aria-label="Quick access">
        <div class="sidebar-heading">SHORTCUTS</div>
        <h2 class="quick-access-title">Quick Access</h2>

        <button class="shortcut-button" data-tool="Projects">
            <span class="shortcut-icon blue"><i class="fa-solid fa-folder-open"></i></span>
            <span class="shortcut-copy">
                <strong>Projects</strong>
                <small>Manage local websites</small>
            </span>
            <i class="fa-solid fa-arrow-up-right-from-square shortcut-arrow"></i>
        </button>

        <button class="shortcut-button" data-tool="Databases">
            <span class="shortcut-icon orange"><i class="fa-solid fa-database"></i></span>
            <span class="shortcut-copy">
                <strong>Databases</strong>
                <small>Manage database instances</small>
            </span>
            <i class="fa-solid fa-arrow-up-right-from-square shortcut-arrow"></i>
        </button>

        <button class="shortcut-button" data-tool="PHP Manager">
            <span class="shortcut-icon purple"><i class="fa-brands fa-php"></i></span>
            <span class="shortcut-copy">
                <strong>PHP Manager</strong>
                <small>Versions and configuration</small>
            </span>
            <i class="fa-solid fa-arrow-up-right-from-square shortcut-arrow"></i>
        </button>

        <button class="shortcut-button" data-tool="Backups">
            <span class="shortcut-icon green"><i class="fa-solid fa-box-archive"></i></span>
            <span class="shortcut-copy">
                <strong>Backups</strong>
                <small>Protect local data</small>
            </span>
            <i class="fa-solid fa-arrow-up-right-from-square shortcut-arrow"></i>
        </button>

        <div class="web-root-card">
            <div class="web-root-heading">
                <i class="fa-solid fa-folder-tree"></i>
                <span>WEB ROOT</span>
            </div>
            <code>D:\NovaHost\home\public</code>
        </div>
    </aside>

</div><script type="module" src="/src/main.ts"></script>
</body>
</html>


========== THEME / CSS CONTROLS ==========

  frontend\public\style.css:2:    font-family: "Segoe UI", Arial, sans-serif;
  frontend\public\style.css:3:    color: #e8edf5;
> frontend\public\style.css:4:    background: #101820;
  frontend\public\style.css:5:    font-synthesis: none;
  frontend\public\style.css:6:    text-rendering: optimizeLegibility;
  frontend\public\style.css:7:    --bg: #101820;
  frontend\public\style.css:25:    min-width: 760px;
  frontend\public\style.css:26:    min-height: 100vh;
> frontend\public\style.css:27:    background: var(--bg);
  frontend\public\style.css:28:    color: var(--text);
  frontend\public\style.css:29:    font-size: 13px;
  frontend\public\style.css:30:}
  frontend\public\style.css:31:
> frontend\public\style.css:32:button {
  frontend\public\style.css:33:    font: inherit;
  frontend\public\style.css:34:}
  frontend\public\style.css:35:
  frontend\public\style.css:37:    width: min(1180px, calc(100% - 32px));
  frontend\public\style.css:38:    margin: 18px auto;
> frontend\public\style.css:39:    background: var(--panel);
  frontend\public\style.css:40:    border: 1px solid var(--border);
  frontend\public\style.css:41:    border-radius: 7px;
  frontend\public\style.css:42:    box-shadow: 0 12px 40px rgba(0, 0, 0, .28);
  frontend\public\style.css:51:    justify-content: space-between;
  frontend\public\style.css:52:    gap: 20px;
> frontend\public\style.css:53:    background: #172530;
  frontend\public\style.css:54:    border-bottom: 1px solid var(--border);
  frontend\public\style.css:55:}
  frontend\public\style.css:56:
  frontend\public\style.css:111:
  frontend\public\style.css:112:.section-title,
> frontend\public\style.css:113:.logs-header {
  frontend\public\style.css:114:    display: flex;
  frontend\public\style.css:115:    align-items: center;
  frontend\public\style.css:116:    justify-content: space-between;
  frontend\public\style.css:145:    height: 36px;
  frontend\public\style.css:146:    padding: 0 12px;
> frontend\public\style.css:147:    background: #263743;
  frontend\public\style.css:148:    color: #c3d0da;
  frontend\public\style.css:149:    font-weight: 650;
  frontend\public\style.css:150:    border-bottom: 1px solid var(--border);
  frontend\public\style.css:162:
  frontend\public\style.css:163:.module-table tbody tr:hover {
> frontend\public\style.css:164:    background: #223440;
  frontend\public\style.css:165:}
  frontend\public\style.css:166:
  frontend\public\style.css:167:.module-table th:first-child,
  frontend\public\style.css:225:
  frontend\public\style.css:226:.btn.start {
> frontend\public\style.css:227:    background: #238b52;
  frontend\public\style.css:228:    border-color: #31a765;
  frontend\public\style.css:229:}
  frontend\public\style.css:230:
  frontend\public\style.css:231:.btn.stop {
> frontend\public\style.css:232:    background: #b83e43;
  frontend\public\style.css:233:    border-color: #d45a5e;
  frontend\public\style.css:234:}
  frontend\public\style.css:235:
  frontend\public\style.css:236:.btn.admin {
> frontend\public\style.css:237:    background: #286d9e;
  frontend\public\style.css:238:    border-color: #3987ba;
  frontend\public\style.css:239:}
  frontend\public\style.css:240:
  frontend\public\style.css:241:.btn.config {
> frontend\public\style.css:242:    background: #586875;
  frontend\public\style.css:243:    border-color: #71818e;
  frontend\public\style.css:244:}
  frontend\public\style.css:245:
  frontend\public\style.css:246:.btn.logs {
> frontend\public\style.css:247:    background: #6653a0;
  frontend\public\style.css:248:    border-color: #8570c2;
  frontend\public\style.css:249:}
  frontend\public\style.css:250:
  frontend\public\style.css:266:    border: 1px solid var(--border);
  frontend\public\style.css:267:    border-radius: 5px;
> frontend\public\style.css:268:    background: #172530;
  frontend\public\style.css:269:}
  frontend\public\style.css:270:
  frontend\public\style.css:271:.tools-section h2 {
  frontend\public\style.css:279:}
  frontend\public\style.css:280:
> frontend\public\style.css:281:.tool-button {
  frontend\public\style.css:282:    min-width: 91px;
  frontend\public\style.css:283:    height: 35px;
  frontend\public\style.css:284:    display: inline-flex;
  frontend\public\style.css:289:    border: 1px solid #405462;
  frontend\public\style.css:290:    border-radius: 4px;
> frontend\public\style.css:291:    background: #263945;
  frontend\public\style.css:292:    color: #dce5ec;
  frontend\public\style.css:293:    cursor: pointer;
  frontend\public\style.css:294:    font-size: 12px;
  frontend\public\style.css:295:}
  frontend\public\style.css:296:
> frontend\public\style.css:297:.tool-button i {
  frontend\public\style.css:298:    color: #65b5ed;
  frontend\public\style.css:299:}
  frontend\public\style.css:300:
> frontend\public\style.css:301:.tool-button:hover {
> frontend\public\style.css:302:    background: #304957;
  frontend\public\style.css:303:    border-color: #5a91b4;
  frontend\public\style.css:304:}
  frontend\public\style.css:305:
  frontend\public\style.css:315:    border-radius: 5px;
  frontend\public\style.css:316:    overflow: hidden;
> frontend\public\style.css:317:    background: #111b23;
  frontend\public\style.css:318:}
  frontend\public\style.css:319:
> frontend\public\style.css:320:.logs-header {
  frontend\public\style.css:321:    margin: 0;
  frontend\public\style.css:322:    padding: 11px 13px;
> frontend\public\style.css:323:    background: #202f3b;
  frontend\public\style.css:324:    border-bottom: 1px solid var(--border);
  frontend\public\style.css:325:}
  frontend\public\style.css:326:
  frontend\public\style.css:330:}
  frontend\public\style.css:331:
> frontend\public\style.css:332:.log-controls button {
  frontend\public\style.css:333:    width: 29px;
  frontend\public\style.css:334:    height: 27px;
  frontend\public\style.css:335:    border: 1px solid #485b68;
  frontend\public\style.css:336:    border-radius: 3px;
> frontend\public\style.css:337:    background: #30424e;
  frontend\public\style.css:338:    color: #dce5ec;
  frontend\public\style.css:339:    cursor: pointer;
  frontend\public\style.css:340:}
  frontend\public\style.css:341:
> frontend\public\style.css:342:.log-controls button:hover {
> frontend\public\style.css:343:    background: #405866;
  frontend\public\style.css:344:}
  frontend\public\style.css:345:
  frontend\public\style.css:346:.log-toolbar {
  frontend\public\style.css:362:    border: 0;
  frontend\public\style.css:363:    border-radius: 3px;
> frontend\public\style.css:364:    background: transparent;
  frontend\public\style.css:365:    color: #9eafbc;
  frontend\public\style.css:366:    cursor: pointer;
  frontend\public\style.css:367:    font-size: 11px;
  frontend\public\style.css:369:
  frontend\public\style.css:370:.filter.active {
> frontend\public\style.css:371:    background: #286d9e;
  frontend\public\style.css:372:    color: white;
  frontend\public\style.css:373:}
  frontend\public\style.css:374:
  frontend\public\style.css:414:    justify-content: space-between;
  frontend\public\style.css:415:    gap: 12px;
> frontend\public\style.css:416:    background: #172530;
  frontend\public\style.css:417:    border-top: 1px solid var(--border);
  frontend\public\style.css:418:    color: #91a4b2;
  frontend\public\style.css:419:    font-size: 11px;
  frontend\public\style.css:447:}
  frontend\public\style.css:448:
> frontend\public\style.css:449:/* NOVAHOST CUSTOM BACKGROUND */
  frontend\public\style.css:450:body {
> frontend\public\style.css:451:    background:
  frontend\public\style.css:452:        linear-gradient(
  frontend\public\style.css:453:            135deg,
  frontend\public\style.css:454:            rgba(5, 12, 24, 0.58),
  frontend\public\style.css:455:            rgba(8, 20, 36, 0.72)
  frontend\public\style.css:456:        ),
> frontend\public\style.css:457:        url("/background.png") center center / cover fixed no-repeat;
  frontend\public\style.css:458:}
  frontend\public\style.css:459:
  frontend\public\style.css:460:.window {
> frontend\public\style.css:461:    background: rgba(15, 29, 43, 0.86);
  frontend\public\style.css:462:    border: 1px solid rgba(95, 165, 220, 0.28);
  frontend\public\style.css:463:    backdrop-filter: blur(14px);
  frontend\public\style.css:464:    -webkit-backdrop-filter: blur(14px);
  frontend\public\style.css:470:/* ==========================================
  frontend\public\style.css:471:   NOVAHOST THREE-PANEL DASHBOARD
> frontend\public\style.css:472:   SIDEBARS + RESPONSIVE LAYOUT + AURORA
  frontend\public\style.css:473:   ========================================== */
  frontend\public\style.css:474:
  frontend\public\style.css:475:body {
  frontend\public\style.css:487:    pointer-events: none;
  frontend\public\style.css:488:    opacity: 0.48;
> frontend\public\style.css:489:    background:
  frontend\public\style.css:490:        radial-gradient(ellipse at 18% 28%,
  frontend\public\style.css:491:            rgba(0, 153, 255, 0.24), transparent 38%),
  frontend\public\style.css:492:        radial-gradient(ellipse at 78% 24%,
  frontend\public\style.css:532:}
  frontend\public\style.css:533:
> frontend\public\style.css:534:/* Shared sidebar appearance */
> frontend\public\style.css:535:.app-shell > .sidebar {
  frontend\public\style.css:536:    box-sizing: border-box;
  frontend\public\style.css:537:    width: 100%;
  frontend\public\style.css:538:    min-width: 0;
  frontend\public\style.css:543:    gap: 18px;
  frontend\public\style.css:544:    color: #dce9f5;
> frontend\public\style.css:545:    background: linear-gradient(
  frontend\public\style.css:546:        160deg,
  frontend\public\style.css:547:        rgba(13, 30, 49, 0.94),
  frontend\public\style.css:548:        rgba(9, 20, 35, 0.91)
  frontend\public\style.css:558:
  frontend\public\style.css:559:/* Left brand */
> frontend\public\style.css:560:.sidebar-brand {
  frontend\public\style.css:561:    display: flex;
  frontend\public\style.css:562:    align-items: center;
  frontend\public\style.css:563:    gap: 11px;
  frontend\public\style.css:566:}
  frontend\public\style.css:567:
> frontend\public\style.css:568:.sidebar-brand-icon {
  frontend\public\style.css:569:    width: 39px;
  frontend\public\style.css:570:    height: 39px;
  frontend\public\style.css:571:    flex: 0 0 39px;
  frontend\public\style.css:576:    border: 1px solid rgba(70, 165, 255, 0.32);
  frontend\public\style.css:577:    border-radius: 10px;
> frontend\public\style.css:578:    background: linear-gradient(
  frontend\public\style.css:579:        145deg,
  frontend\public\style.css:580:        rgba(30, 130, 220, 0.25),
  frontend\public\style.css:581:        rgba(65, 75, 190, 0.15)
  frontend\public\style.css:584:}
  frontend\public\style.css:585:
> frontend\public\style.css:586:.sidebar-brand-text {
  frontend\public\style.css:587:    min-width: 0;
  frontend\public\style.css:588:    display: flex;
  frontend\public\style.css:589:    flex-direction: column;
  frontend\public\style.css:591:}
  frontend\public\style.css:592:
> frontend\public\style.css:593:.sidebar-brand-text strong {
  frontend\public\style.css:594:    color: #f0f7ff;
  frontend\public\style.css:595:    font-size: 15px;
  frontend\public\style.css:596:    letter-spacing: 0.3px;
  frontend\public\style.css:597:}
  frontend\public\style.css:598:
> frontend\public\style.css:599:.sidebar-brand-text span {
  frontend\public\style.css:600:    color: #7898b4;
  frontend\public\style.css:601:    font-size: 9px;
  frontend\public\style.css:602:    letter-spacing: 1.5px;
  frontend\public\style.css:604:
  frontend\public\style.css:605:/* Vertical navigation groups */
> frontend\public\style.css:606:.sidebar-group {
  frontend\public\style.css:607:    display: flex;
  frontend\public\style.css:608:    flex-direction: column;
  frontend\public\style.css:609:    gap: 6px;
  frontend\public\style.css:610:}
  frontend\public\style.css:611:
> frontend\public\style.css:612:.sidebar-heading {
  frontend\public\style.css:613:    margin: 0 0 5px;
  frontend\public\style.css:614:    padding: 0 9px;
  frontend\public\style.css:615:    color: #7894ad;
  frontend\public\style.css:619:}
  frontend\public\style.css:620:
> frontend\public\style.css:621:.sidebar-link {
  frontend\public\style.css:622:    box-sizing: border-box;
  frontend\public\style.css:623:    width: 100%;
  frontend\public\style.css:624:    min-height: 42px;
  frontend\public\style.css:630:    border: 1px solid transparent;
  frontend\public\style.css:631:    border-radius: 7px;
> frontend\public\style.css:632:    background: transparent;
  frontend\public\style.css:633:    color: #a9bdcf;
  frontend\public\style.css:634:    text-align: left;
  frontend\public\style.css:635:    font: inherit;
  frontend\public\style.css:637:    cursor: pointer;
  frontend\public\style.css:638:    transition:
> frontend\public\style.css:639:        background 180ms ease,
  frontend\public\style.css:640:        border-color 180ms ease,
  frontend\public\style.css:641:        color 180ms ease,
  frontend\public\style.css:642:        transform 180ms ease;
  frontend\public\style.css:643:}
  frontend\public\style.css:644:
> frontend\public\style.css:645:.sidebar-link i {
  frontend\public\style.css:646:    width: 18px;
  frontend\public\style.css:647:    flex: 0 0 18px;
  frontend\public\style.css:648:    color: #7fa8c9;
  frontend\public\style.css:651:}
  frontend\public\style.css:652:
> frontend\public\style.css:653:.sidebar-link:hover {
  frontend\public\style.css:654:    color: #ffffff;
> frontend\public\style.css:655:    background: rgba(54, 133, 195, 0.13);
  frontend\public\style.css:656:    border-color: rgba(88, 166, 230, 0.18);
  frontend\public\style.css:657:    transform: translateX(2px);
  frontend\public\style.css:658:}
  frontend\public\style.css:659:
> frontend\public\style.css:660:.sidebar-link.active {
  frontend\public\style.css:661:    color: #ffffff;
> frontend\public\style.css:662:    background: linear-gradient(
  frontend\public\style.css:663:        90deg,
  frontend\public\style.css:664:        rgba(30, 132, 220, 0.24),
  frontend\public\style.css:665:        rgba(30, 132, 220, 0.07)
  frontend\public\style.css:669:}
  frontend\public\style.css:670:
> frontend\public\style.css:671:.sidebar-link.active i {
  frontend\public\style.css:672:    color: #65bdff;
  frontend\public\style.css:673:}
  frontend\public\style.css:674:
  frontend\public\style.css:675:/* Local environment status */
> frontend\public\style.css:676:.sidebar-status {
  frontend\public\style.css:677:    margin-top: auto;
  frontend\public\style.css:678:    padding: 12px 9px 2px;
  frontend\public\style.css:679:    display: flex;
  frontend\public\style.css:686:}
  frontend\public\style.css:687:
> frontend\public\style.css:688:.sidebar-status-dot {
  frontend\public\style.css:689:    width: 8px;
  frontend\public\style.css:690:    height: 8px;
  frontend\public\style.css:691:    flex: 0 0 8px;
  frontend\public\style.css:692:    border-radius: 50%;
> frontend\public\style.css:693:    background: #35d69a;
  frontend\public\style.css:694:    box-shadow: 0 0 10px rgba(53, 214, 154, 0.65);
  frontend\public\style.css:695:}
  frontend\public\style.css:696:
  frontend\public\style.css:703:}
  frontend\public\style.css:704:
> frontend\public\style.css:705:.shortcut-button {
  frontend\public\style.css:706:    box-sizing: border-box;
  frontend\public\style.css:707:    width: 100%;
  frontend\public\style.css:708:    min-height: 66px;
  frontend\public\style.css:713:    border: 1px solid rgba(100, 150, 195, 0.13);
  frontend\public\style.css:714:    border-radius: 9px;
> frontend\public\style.css:715:    background: rgba(23, 43, 63, 0.56);
  frontend\public\style.css:716:    color: #e5f0fb;
  frontend\public\style.css:717:    text-align: left;
  frontend\public\style.css:718:    cursor: pointer;
  frontend\public\style.css:719:    transition:
> frontend\public\style.css:720:        background 180ms ease,
  frontend\public\style.css:721:        border-color 180ms ease,
  frontend\public\style.css:722:        transform 180ms ease;
  frontend\public\style.css:723:}
  frontend\public\style.css:724:
> frontend\public\style.css:725:.shortcut-button:hover {
> frontend\public\style.css:726:    background: rgba(37, 75, 110, 0.65);
  frontend\public\style.css:727:    border-color: rgba(82, 166, 235, 0.38);
  frontend\public\style.css:728:    transform: translateY(-2px);
  frontend\public\style.css:729:}
  frontend\public\style.css:741:.shortcut-icon.blue {
  frontend\public\style.css:742:    color: #6bc0ff;
> frontend\public\style.css:743:    background: rgba(45, 145, 235, 0.16);
  frontend\public\style.css:744:}
  frontend\public\style.css:745:
  frontend\public\style.css:746:.shortcut-icon.orange {
  frontend\public\style.css:747:    color: #ffb66d;
> frontend\public\style.css:748:    background: rgba(235, 145, 45, 0.15);
  frontend\public\style.css:749:}
  frontend\public\style.css:750:
  frontend\public\style.css:751:.shortcut-icon.purple {
  frontend\public\style.css:752:    color: #c3a0ff;
> frontend\public\style.css:753:    background: rgba(145, 95, 235, 0.16);
  frontend\public\style.css:754:}
  frontend\public\style.css:755:
  frontend\public\style.css:756:.shortcut-icon.green {
  frontend\public\style.css:757:    color: #65e0ae;
> frontend\public\style.css:758:    background: rgba(45, 195, 135, 0.15);
  frontend\public\style.css:759:}
  frontend\public\style.css:760:
  frontend\public\style.css:761:.shortcut-copy {
  frontend\public\style.css:793:    border: 1px solid rgba(85, 155, 210, 0.2);
  frontend\public\style.css:794:    border-radius: 9px;
> frontend\public\style.css:795:    background: rgba(12, 30, 48, 0.72);
  frontend\public\style.css:796:}
  frontend\public\style.css:797:
  frontend\public\style.css:798:.web-root-heading {
  frontend\public\style.css:833:    }
  frontend\public\style.css:834:
> frontend\public\style.css:835:    .sidebar-link,
> frontend\public\style.css:836:    .shortcut-button {
  frontend\public\style.css:837:        transition: none;
  frontend\public\style.css:838:    }
  frontend\public\style.css:839:}
  frontend\public\style.css:851:
  frontend\public\style.css:852:.module-table .btn.stop {
> frontend\public\style.css:853:    background: #7650c8;
  frontend\public\style.css:854:    border-color: #7650c8;
  frontend\public\style.css:855:    color: #fff;
  frontend\public\style.css:856:}
  frontend\public\style.css:857:
  frontend\public\style.css:858:.module-table .btn.stop:hover {
> frontend\public\style.css:859:    background: #6540b5;
  frontend\public\style.css:860:}

========== CSS START ==========
:root {
    font-family: "Segoe UI", Arial, sans-serif;
    color: #e8edf5;
    background: #101820;
    font-synthesis: none;
    text-rendering: optimizeLegibility;
    --bg: #101820;
    --panel: #192631;
    --panel2: #202f3b;
    --border: #344653;
    --text: #e8edf5;
    --muted: #9aabb8;
    --blue: #2588d8;
    --green: #27ae60;
    --red: #d94b4b;
    --orange: #e59b36;
}

* {
    box-sizing: border-box;
}

body {
    margin: 0;
    min-width: 760px;
    min-height: 100vh;
    background: var(--bg);
    color: var(--text);
    font-size: 13px;
}

button {
    font: inherit;
}

.window {
    width: min(1180px, calc(100% - 32px));
    margin: 18px auto;
    background: var(--panel);
    border: 1px solid var(--border);
    border-radius: 7px;
    box-shadow: 0 12px 40px rgba(0, 0, 0, .28);
    overflow: hidden;
}

.topbar {
    min-height: 58px;
    padding: 0 18px;
    display: flex;
    align-items: center;
    justify-content: space-between;
    gap: 20px;
    background: #172530;
    border-bottom: 1px solid var(--border);
}

.brand,
.top-info,
.top-info span {
    display: flex;
    align-items: center;
}

.brand {
    gap: 10px;
}

.brand > i {
    color: #47a8f0;
    font-size: 23px;
}

.brand strong {
    font-size: 19px;
    letter-spacing: .3px;
}

.brand > span {
    color: var(--muted);
    border-left: 1px solid var(--border);
    padding-left: 12px;
    margin-left: 3px;
}

.top-info {
    gap: 20px;
    color: var(--muted);
    font-size: 12px;
}

.engine-status {
    color: #e4a34a;
    font-weight: 700;
    letter-spacing: .4px;
}

#clock,
#footer-clock,
.mono {
    font-family: Consolas, monospace;
    font-variant-numeric: tabular-nums;
}

main {
    padding: 18px;
}

section {
    margin-bottom: 18px;
}

.section-title,
.logs-header {
    display: flex;
    align-items: center;
    justify-content: space-between;
    margin-bottom: 10px;
}

h2 {
    margin: 0;
    font-size: 15px;
    font-weight: 650;
}

.section-title > span {
    color: var(--muted);
    font-size: 12px;
}

.table-wrap {
    overflow-x: auto;
    border: 1px solid var(--border);
    border-radius: 5px;
}

.module-table {
    width: 100%;
    border-collapse: collapse;
    text-align: left;
    white-space: nowrap;
}

.module-table th {
    height: 36px;
    padding: 0 12px;
    background: #263743;
    color: #c3d0da;
    font-weight: 650;
    border-bottom: 1px solid var(--border);
}

.module-table td {
    height: 49px;
    padding: 6px 12px;
    border-bottom: 1px solid #2b3b47;
}

.module-table tbody tr:last-child td {
    border-bottom: 0;
}

.module-table tbody tr:hover {
    background: #223440;
}

.module-table th:first-child,
.module-table td:first-child {
    min-width: 220px;
}

.module-table th:nth-child(2) {
    width: 100px;
}

.module-table th:nth-child(3) {
    width: 110px;
}

.module-table th:last-child {
