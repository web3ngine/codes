========== main.go ==========
package main

import (
        "embed"

        "log"
        "time"

        "github.com/wailsapp/wails/v3/pkg/application"
)

// Wails uses Go's `embed` package to embed the frontend files into the binary.
// Any files in the frontend/dist folder will be embedded into the binary and
// made available to the frontend.
// See https://pkg.go.dev/embed for more information.

//go:embed all:frontend/dist
var assets embed.FS

func init() {
        // Register a custom event whose associated data type is string.
        // This is not required, but the binding generator will pick up registered events
        // and provide a strongly typed JS/TS API for them.
        application.RegisterEvent[string]("time")
}

// main function serves as the application's entry point. It initializes the application, creates a window,
// and starts a goroutine that emits a time-based event every second. It subsequently runs the application and
// logs any error that might occur.
func main() {

        // Create a new Wails application by providing the necessary options.
        // Variables 'Name' and 'Description' are for application metadata.
        // 'Assets' configures the asset server with the 'FS' variable pointing to the frontend files.
        // 'Bind' is a list of Go struct instances. The frontend has access to the methods of these instances.
        // 'Mac' options tailor the application when running an macOS.
        app := application.New(application.Options{
                Name:        "NovaHost",
                Description: "A demo of using raw HTML & CSS",
                Services: []application.Service{
                        application.NewService(&GreetService{}),
                },
                Assets: application.AssetOptions{
                        Handler: application.AssetFileServerFS(assets),
                },
                Mac: application.MacOptions{
                        ApplicationShouldTerminateAfterLastWindowClosed: true,
                },
        })

        // Create a new window with the necessary options.
        // 'Title' is the title of the window.
        // 'Mac' options tailor the window when running on macOS.
        // 'BackgroundColour' is the background colour of the window.
        // 'URL' is the URL that will be loaded into the webview.
        app.Window.NewWithOptions(application.WebviewWindowOptions{
                Title: "NovaHost Local Hosting",
                // Window sized to the golden ratio (1000 / 618 â‰ˆ 1.618).
                Width:  1000,
                Height: 618,
                Mac: application.MacWindow{
                        InvisibleTitleBarHeight: 50,
                        Backdrop:                application.MacBackdropTranslucent,
                        TitleBar:                application.MacTitleBarHiddenInset,
                },
                BackgroundColour: application.NewRGB(6, 7, 15),
                URL:              "/",
        })

        // Create a goroutine that emits an event containing the current time every second.
        // The frontend can listen to this event and update the UI accordingly.
        go func() {
                for {
                        now := time.Now().Format(time.RFC1123)
                        app.Event.Emit("time", now)
                        time.Sleep(time.Second)
                }
        }()

        // Run the application. This blocks until the application has been exited.
        err := app.Run()

        // If an error occurred while running the application, log it and exit.
        if err != nil {
                log.Fatal(err)
        }
}


========== greetservice.go ==========
package main

type GreetService struct{}

func (g *GreetService) Greet(name string) string {
        return "Hello " + name + "!"
}

========== go.mod ==========
module novahost.local/desktop

go 1.26.0

require github.com/wailsapp/wails/v3 v3.0.0

require (
        github.com/adrg/xdg v0.5.3 // indirect
        github.com/coder/websocket v1.8.14 // indirect
        github.com/go-ole/go-ole v1.3.0 // indirect
        github.com/godbus/dbus/v5 v5.2.2 // indirect
        github.com/mattn/go-colorable v0.1.14 // indirect
        github.com/mattn/go-isatty v0.0.20 // indirect
        golang.org/x/sys v0.48.0 // indirect
)

========== frontend\src\main.ts ==========
import "@fortawesome/fontawesome-free/css/all.min.css";

type LogType = "success" | "stop" | "error" | "info";

interface LogEntry {
    time: string;
    type: LogType;
    module: string;
    message: string;
}

let logs: LogEntry[] = [];
let logsPaused = false;
let activeFilter = "all";

const $ = <T extends HTMLElement>(selector: string): T | null =>
    document.querySelector<T>(selector);

const $$ = <T extends HTMLElement>(selector: string): T[] =>
    Array.from(document.querySelectorAll<T>(selector));

function updateClock(): void {
    const now = new Date();
    const systemTimeZone = Intl.DateTimeFormat().resolvedOptions().timeZone;

    const clock = new Intl.DateTimeFormat("bg-BG", {
        timeZone: systemTimeZone,
        hour: "2-digit",
        minute: "2-digit",
        second: "2-digit",
        hour12: false
    }).format(now);

    const date = new Intl.DateTimeFormat("bg-BG", {
        timeZone: systemTimeZone,
        day: "2-digit",
        month: "2-digit",
        year: "numeric"
    }).format(now);

    const topClock = $("#clock");
    const footerClock = $("#footer-clock");
    const timezoneLabel = $("#timezone-label");

    if (topClock) topClock.textContent = clock;
    if (timezoneLabel) timezoneLabel.textContent = systemTimeZone;
    if (footerClock) footerClock.textContent = `${date} ${clock}`;
}

function renderLogs(): void {
    const list = $("#log-list");
    const count = $("#log-count");

    if (!list) return;

    const filtered = logs.filter(log =>
        activeFilter === "all" || log.type === activeFilter
    );

    if (count) {
        count.textContent = `${filtered.length} entries`;
    }

    if (filtered.length === 0) {
        list.innerHTML = '<div class="log-empty">ÐÑÐ¼Ð° Ð½Ð°Ð»Ð¸Ñ‡Ð½Ð¸ Ð»Ð¾Ð³Ð¾Ð²Ðµ. Ð ÐµÐ°Ð»Ð½Ð¸Ñ‚Ðµ ÑÑŠÐ±Ð¸Ñ‚Ð¸Ñ Ñ‰Ðµ ÑÐµ Ð¿Ð¾ÐºÐ°Ð·Ð²Ð°Ñ‚ Ñ‚ÑƒÐº.</div>';
        return;
    }

    list.replaceChildren();

    for (const log of filtered) {
        const row = document.createElement("div");
        row.className = "log-row";

        const time = document.createElement("span");
        time.className = "log-time";
        time.textContent = log.time;

        const type = document.createElement("span");
        type.className = `tag-${log.type}`;
        type.textContent = log.type.toUpperCase();

        const module = document.createElement("span");
        module.className = "log-module";
        module.textContent = log.module;

        const message = document.createElement("span");
        message.className = "log-message";
        message.textContent = log.message;

        row.append(time, type, module, message);
        list.appendChild(row);
    }
}

function addLog(
    type: LogType,
    module: string,
    message: string
): void {
    if (logsPaused) return;

    logs.push({
        time: new Date().toLocaleTimeString("bg-BG", {
            hour12: false
        }),
        type,
        module,
        message
    });

    renderLogs();
}

function setupLogControls(): void {
    $$(".filter").forEach(button => {
        button.addEventListener("click", () => {
            activeFilter = button.dataset.filter || "all";

            $$(".filter").forEach(item =>
                item.classList.toggle("active", item === button)
            );

            renderLogs();
        });
    });

    $("#pause-logs")?.addEventListener("click", event => {
        logsPaused = !logsPaused;

        const button = event.currentTarget as HTMLButtonElement;
        button.innerHTML = logsPaused
            ? '<i class="fa-solid fa-play"></i>'
            : '<i class="fa-solid fa-pause"></i>';

        button.title = logsPaused ? "Resume logs" : "Pause logs";
    });

    $("#clear-logs")?.addEventListener("click", () => {
        logs = [];
        renderLogs();
    });

    $("#export-logs")?.addEventListener("click", () => {
        if (logs.length === 0) {
            alert("ÐÑÐ¼Ð° Ð»Ð¾Ð³Ð¾Ð²Ðµ Ð·Ð° ÐµÐºÑÐ¿Ð¾Ñ€Ñ‚Ð¸Ñ€Ð°Ð½Ðµ.");
            return;
        }

        const content = [
            "Time\tType\tModule\tMessage",
            ...logs.map(log =>
                `${log.time}\t${log.type}\t${log.module}\t${log.message}`
            )
        ].join("\n");

        const blob = new Blob([content], {
            type: "text/plain;charset=utf-8"
        });

        const url = URL.createObjectURL(blob);
        const link = document.createElement("a");

        link.href = url;
        link.download = `novahost-logs-${Date.now()}.txt`;
        link.click();

        URL.revokeObjectURL(url);
    });

    renderLogs();
}

function setupTools(): void {
    const message = $("#tool-message");

    $$<HTMLButtonElement>(".tool-button").forEach(button => {
        button.addEventListener("click", () => {
            const tool = button.dataset.tool || button.textContent?.trim() || "Tool";

            if (message) {
                message.textContent =
                    `${tool}: Ð¼Ð¾Ð´ÑƒÐ»ÑŠÑ‚ Ñ‰Ðµ Ð±ÑŠÐ´Ðµ ÑÐ²ÑŠÑ€Ð·Ð°Ð½ Ñ Ñ€ÐµÐ°Ð»Ð½Ð°Ñ‚Ð° Ñ„ÑƒÐ½ÐºÑ†Ð¸Ð¾Ð½Ð°Ð»Ð½Ð¾ÑÑ‚ Ð² ÑÐ»ÐµÐ´Ð²Ð°Ñ‰Ð¸Ñ ÐµÑ‚Ð°Ð¿.`;
            }
        });
    });
}

function setupModuleButtons(): void {
    // Start/Stop/Admin/Config/Logs Ð¾ÑÑ‚Ð°Ð²Ð°Ñ‚ Ð¸Ð·ÐºÐ»ÑŽÑ‡ÐµÐ½Ð¸,
    // Ð´Ð¾ÐºÐ°Ñ‚Ð¾ Ð½Ðµ Ð±ÑŠÐ´Ð°Ñ‚ ÑÐ²ÑŠÑ€Ð·Ð°Ð½Ð¸ Ñ Ñ€ÐµÐ°Ð»Ð½Ð¸Ñ Go backend.
    $$(".module-table button").forEach(button => {
        if (
            button.classList.contains("start") ||
            button.classList.contains("stop") ||
            button.classList.contains("admin") ||
            button.classList.contains("config") ||
            button.classList.contains("logs")
        ) {
            (button as HTMLButtonElement).disabled = true;
        }
    });
}

function initialize(): void {
    updateClock();
    window.setInterval(updateClock, 1000);

    setupLogControls();
    setupTools();
    setupModuleButtons();

    console.info("NovaHost frontend initialized.");
}

if (document.readyState === "loading") {
    document.addEventListener("DOMContentLoaded", initialize);
} else {
    initialize();
}

========== frontend\index.html ==========
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
<div class="window">
    <header class="topbar">
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
        <section class="module-section">
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

<script type="module" src="/src/main.ts"></script>
</body>
</html>

========== frontend\public\style.css ==========
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
    width: 350px;
}

.module-icon {
    width: 23px;
    margin-right: 7px;
    text-align: center;
    font-size: 15px;
}

.blue { color: #49a8ef; }
.purple { color: #b48af0; }
.orange { color: #e9a34b; }
.red { color: #ed6b6b; }
.green { color: #55c98a; }
.yellow { color: #e5cb56; }

.mono {
    color: #c1ced8;
    font-size: 12px;
}

.actions {
    display: flex;
    align-items: center;
    gap: 5px;
}

.btn {
    min-width: 65px;
    height: 27px;
    padding: 0 9px;
    border: 1px solid transparent;
    border-radius: 3px;
    color: white;
    font-size: 11px;
    font-weight: 700;
    cursor: pointer;
}

.btn:disabled {
    opacity: .48;
    cursor: not-allowed;
}

.btn.start {
    background: #238b52;
    border-color: #31a765;
}

.btn.stop {
    background: #b83e43;
    border-color: #d45a5e;
}

.btn.admin {
    background: #286d9e;
    border-color: #3987ba;
}

.btn.config {
    background: #586875;
    border-color: #71818e;
}

.btn.logs {
    background: #6653a0;
    border-color: #8570c2;
}

.table-note {
    display: flex;
    align-items: center;
    gap: 8px;
    margin-top: 9px;
    color: var(--muted);
    font-size: 11px;
}

.table-note i {
    color: #e5b34e;
}

.tools-section {
    padding: 13px;
    border: 1px solid var(--border);
    border-radius: 5px;
    background: #172530;
}

.tools-section h2 {
    margin-bottom: 11px;
}

.tools-grid {
    display: flex;
    flex-wrap: wrap;
    gap: 7px;
}

.tool-button {
    min-width: 91px;
    height: 35px;
    display: inline-flex;
    align-items: center;
    justify-content: center;
    gap: 7px;
    padding: 0 10px;
    border: 1px solid #405462;
    border-radius: 4px;
    background: #263945;
    color: #dce5ec;
    cursor: pointer;
    font-size: 12px;
}

.tool-button i {
    color: #65b5ed;
}

.tool-button:hover {
    background: #304957;
    border-color: #5a91b4;
}

.tool-message {
    min-height: 28px;
    padding-top: 10px;
    color: var(--muted);
    font-size: 11px;
}

.logs-section {
    border: 1px solid var(--border);
    border-radius: 5px;
    overflow: hidden;
    background: #111b23;
}

.logs-header {
    margin: 0;
    padding: 11px 13px;
    background: #202f3b;
    border-bottom: 1px solid var(--border);
}

.log-controls {
    display: flex;
    gap: 5px;
}

.log-controls button {
    width: 29px;
    height: 27px;
    border: 1px solid #485b68;
    border-radius: 3px;
    background: #30424e;
    color: #dce5ec;
    cursor: pointer;
}

.log-controls button:hover {
    background: #405866;
}

.log-toolbar {
    min-height: 39px;
    padding: 5px 10px;
    display: flex;
    align-items: center;
    justify-content: space-between;
    border-bottom: 1px solid #283944;
}

.filters {
    display: flex;
    gap: 4px;
}

.filter {
    padding: 5px 10px;
    border: 0;
    border-radius: 3px;
    background: transparent;
    color: #9eafbc;
    cursor: pointer;
    font-size: 11px;
}

.filter.active {
    background: #286d9e;
    color: white;
}

#log-count {
    color: var(--muted);
    font-size: 11px;
}

.log-list {
    height: 175px;
    overflow: auto;
    padding: 10px 12px;
    font-family: Consolas, monospace;
    font-size: 11px;
}

.log-empty {
    padding: 12px 2px;
    color: #8295a3;
}

.log-row {
    display: grid;
    grid-template-columns: 80px 65px 120px 1fr;
    gap: 10px;
    padding: 5px 0;
    border-bottom: 1px solid #1e2c35;
}

.log-time { color: #8498a7; }
.log-module { color: #67b4e8; }
.log-message { color: #c7d2da; }
.tag-success { color: #51ca85; }
.tag-stop { color: #b89af2; }
.tag-error { color: #ed6868; }
.tag-info { color: #66b4eb; }

footer {
    min-height: 37px;
    padding: 0 18px;
    display: flex;
    align-items: center;
    justify-content: space-between;
    gap: 12px;
    background: #172530;
    border-top: 1px solid var(--border);
    color: #91a4b2;
    font-size: 11px;
}

@media (max-width: 850px) {
    .window {
        width: calc(100% - 12px);
        margin: 6px auto;
    }

    main {
        padding: 10px;
    }

    .topbar,
    footer {
        padding-left: 10px;
        padding-right: 10px;
    }

    .top-info {
        gap: 8px;
    }

    footer {
        flex-wrap: wrap;
        padding-top: 8px;
        padding-bottom: 8px;
    }
}

/* NOVAHOST CUSTOM BACKGROUND */
body {
    background:
        linear-gradient(
            135deg,
            rgba(5, 12, 24, 0.58),
            rgba(8, 20, 36, 0.72)
        ),
        url("/background.png") center center / cover fixed no-repeat;
}

.window {
    background: rgba(15, 29, 43, 0.86);
    border: 1px solid rgba(95, 165, 220, 0.28);
    backdrop-filter: blur(14px);
    -webkit-backdrop-filter: blur(14px);
    box-shadow: 0 0 35px rgba(0, 130, 255, 0.12),
                0 20px 70px rgba(0, 0, 0, 0.45);
}


========== frontend\package.json ==========
{
  "name": "frontend",
  "private": true,
  "version": "0.0.0",
  "type": "module",
  "scripts": {
    "dev": "vite",
    "build:dev": "vite build --minify false --mode development",
    "build": "vite build --mode production",
    "preview": "vite preview"
  },
  "dependencies": {
    "@fortawesome/fontawesome-free": "^7.3.1",
    "@wailsio/runtime": "3.0.0"
  },
  "devDependencies": {
    "typescript": "^4.9.3",
    "vite": "^8.2.4"
  }
}

========== build\config.yml ==========
# This file contains the configuration for this project.
# When you update `info` or `fileAssociations`, run `wails3 task common:update:build-assets` to update the assets.
# Note that this will overwrite any changes you have made to the assets.
version: '3'

# This information is used to generate the build assets.
info:
  companyName: "NovaHost" # The name of the company
  productName: "NovaHost Desktop" # The name of the application
  productIdentifier: "com.novahost.desktop" # The unique product identifier
  description: "NovaHost Local Server Management Platform" # The application description
  copyright: "(c) 2026, NovaHost" # Copyright text
  comments: "NovaHost Desktop Application" # Comments
  version: "0.1.0" # The application version
  # cfBundleIconName: "appicon" # The macOS icon name in Assets.car icon bundles (optional)
  #                               # Should match the name of your .icon file without the extension
  #                               # If not set and Assets.car exists, defaults to "appicon"

# iOS build configuration (uncomment to customise iOS project generation)
# Note: Keys under `ios` OVERRIDE values under `info` when set.
# ios:
#   # The iOS bundle identifier used in the generated Xcode project (CFBundleIdentifier)
#   bundleID: "com.mycompany.myproduct"
#   # The display name shown under the app icon (CFBundleDisplayName/CFBundleName)
#   displayName: "My Product"
#   # The app version to embed in Info.plist (CFBundleShortVersionString/CFBundleVersion)
#   version: "0.1.0"
#   # The company/organisation name for templates and project settings
#   company: "My Company"
#   # Additional comments to embed in Info.plist metadata
#   comments: "NovaHost Desktop Application"

# Dev mode configuration
dev_mode:
  root_path: .
  log_level: warn
  debounce: 1000
  ignore:
    dir:
      - .git
      - node_modules
      - frontend
      - bin
    file:
      - .DS_Store
      - .gitignore
      - .gitkeep
      - "*_test.go"
    watched_extension:
      - "*.go"
      - "*.js" # Watch for changes to JS/TS files included using the //wails:include directive.
      - "*.ts" # The frontend directory will be excluded entirely by the setting above.
    git_ignore: true
  executes:
    - cmd: wails3 build DEV=true
      type: blocking
    - cmd: wails3 task common:dev:frontend
      type: background
      # The standard frontend task waits up to 60s for HTTP 200 from
      # FRONTEND_DEVSERVER_URL before starting the app. Override if needed;
      # custom background commands can opt in with the same readiness block.
      # readiness:
      #   http: "http://localhost:5173/"
      #   timeout: "60s"
      #   interval: "200ms"
    - cmd: wails3 task run
      type: primary
      exit_policy: shutdown

# File Associations
# More information at: https://v3.wails.io/noit/done/yet
fileAssociations:
#  - ext: wails
#    name: Wails
#    description: Wails Application File
#    iconName: wailsFileIcon
#    role: Editor
#  - ext: jpg
#    name: JPEG
#    description: Image File
#    iconName: jpegFileIcon
#    role: Editor
#    mimeType: image/jpeg  # (optional)

# Other data
other:
  - name: My Other Data

========== build\windows\Taskfile.yml ==========
version: '3'

includes:
  common: ../Taskfile.yml

vars:
  # Signing configuration - edit these values for your project
  # SIGN_CERTIFICATE: "path/to/certificate.pfx"
  # SIGN_THUMBPRINT: "certificate-thumbprint"  # Alternative to SIGN_CERTIFICATE
  # TIMESTAMP_SERVER: "http://timestamp.digicert.com"
  #
  # Password is stored securely in system keychain. Run: wails3 setup signing

  # Docker image for cross-compilation with CGO (used when CGO_ENABLED=1 on non-Windows)
  CROSS_IMAGE: wails-cross

tasks:
  build:
    summary: Builds the application for Windows
    cmds:
      # Auto-detect CGO: if CGO_ENABLED=1, use Docker; otherwise use native Go cross-compile
      - task: '{{if and (ne OS "windows") (eq .CGO_ENABLED "1")}}build:docker{{else}}build:native{{end}}'
        vars:
          ARCH: '{{.ARCH}}'
          DEV: '{{.DEV}}'
          EXTRA_TAGS: '{{.EXTRA_TAGS}}'
          OBFUSCATED: '{{.OBFUSCATED}}'
          GARBLE_ARGS: '{{.GARBLE_ARGS}}'
    vars:
      # Default to CGO_ENABLED=0 if not explicitly set
      CGO_ENABLED: '{{.CGO_ENABLED | default "0"}}'

  build:native:
    summary: Builds the application using native Go cross-compilation
    internal: true
    deps:
      - task: common:go:mod:tidy
      - task: common:build:frontend
        vars:
          BUILD_FLAGS:
            ref: .BUILD_FLAGS
          OBFUSCATED:
            ref: .OBFUSCATED
          DEV:
            ref: .DEV
      - task: common:generate:icons
    preconditions:
      - sh: '{{if eq .OBFUSCATED "true"}}command -v garble >/dev/null 2>&1{{else}}true{{end}}'
        msg: "garble is required for obfuscated builds. Install it with: go install mvdan.cc/garble@v0.16.0 (requires Go 1.24+). See https://github.com/burrowers/garble/releases for version/toolchain compatibility."
    cmds:
      - task: generate:syso
        vars:
          ARCH: '{{.ARCH}}'
      - '{{if eq .OBFUSCATED "true"}}garble {{.GARBLE_ARGS}} build{{else}}go build{{end}} {{.BUILD_FLAGS}} -o "{{.BIN_DIR}}/{{.APP_NAME}}.exe"'
      - cmd: powershell Remove-item *.syso
        platforms: [windows]
      - cmd: rm -f *.syso
        platforms: [linux, darwin]
    vars:
      BUILD_FLAGS: '{{if eq .DEV "true"}}{{if or .EXTRA_TAGS (eq .OBFUSCATED "true")}}-tags {{if eq .OBFUSCATED "true"}}wails_obfuscated{{if .EXTRA_TAGS}},{{end}}{{end}}{{.EXTRA_TAGS}} {{end}}-buildvcs=false -gcflags=all="-l"{{else}}-tags production{{if eq .OBFUSCATED "true"}},wails_obfuscated{{end}}{{if .EXTRA_TAGS}},{{.EXTRA_TAGS}}{{end}} -trimpath -buildvcs=false -ldflags="-w -s -H windowsgui"{{end}}'
    env:
      GOOS: windows
      CGO_ENABLED: '{{.CGO_ENABLED | default "0"}}'
      GOARCH: '{{.ARCH | default ARCH}}'

  build:docker:
    summary: Cross-compiles for Windows using Docker with Zig (for CGO builds on non-Windows)
    internal: true
    deps:
      - task: common:build:frontend
        vars:
          OBFUSCATED:
            ref: .OBFUSCATED
      - task: common:generate:icons
    preconditions:
      - sh: docker info > /dev/null 2>&1
        msg: "Docker is required for CGO cross-compilation. Please install Docker."
      - sh: docker image inspect {{.CROSS_IMAGE}} > /dev/null 2>&1
        msg: |
          Docker image '{{.CROSS_IMAGE}}' not found.
          Build it first: wails3 task setup:docker
    cmds:
      - task: generate:syso
        vars:
          ARCH: '{{.ARCH}}'
      - docker run --rm -v "{{.ROOT_DIR}}:/app" {{.DOCKER_MOUNTS}} -e APP_NAME="{{.APP_NAME}}" {{if .EXTRA_TAGS}}-e EXTRA_TAGS="{{.EXTRA_TAGS}}"{{end}} {{if eq .OBFUSCATED "true"}}-e OBFUSCATED=true{{end}} {{if .GARBLE_ARGS}}-e GARBLE_ARGS="{{.GARBLE_ARGS}}"{{end}} {{.CROSS_IMAGE}} windows {{.DOCKER_ARCH}}
      - cmd: docker run --rm -v "{{.ROOT_DIR}}:/app" alpine chown -R $(id -u):$(id -g) /app/bin
        platforms: [linux, darwin]
      - rm -f *.syso
    vars:
      DOCKER_ARCH: '{{.ARCH | default "amd64"}}'
      # Generate Docker volume mounts: Go module cache + go.mod replace directives
      # Uses wails3 tool docker-mounts for cross-platform compatibility (Windows/Linux/macOS)
      DOCKER_MOUNTS:
        sh: 'wails3 tool docker-mounts'

  package:
    summary: Packages the application
    cmds:
      - task: '{{if eq (.FORMAT | default "nsis") "msix"}}create:msix:package{{else}}create:nsis:installer{{end}}'
        vars:
          INSTALL_SCOPE: '{{.INSTALL_SCOPE | default "machine"}}'
    vars:
      FORMAT: '{{.FORMAT | default "nsis"}}'
      INSTALL_SCOPE: '{{.INSTALL_SCOPE | default "machine"}}'

  generate:syso:
    summary: Generates Windows `.syso` file
    dir: build
    cmds:
      - wails3 generate syso -arch {{.ARCH}} -icon windows/icon.ico -manifest windows/wails.exe.manifest -info windows/info.json -out ../wails_windows_{{.ARCH}}.syso
    vars:
      ARCH: '{{.ARCH | default ARCH}}'

  create:nsis:installer:
    summary: Creates an NSIS installer
    dir: build/windows/nsis
    deps:
      - task: build
    preconditions:
      - sh: '[ "{{.INSTALL_SCOPE}}" = "user" ] || [ "{{.INSTALL_SCOPE}}" = "machine" ]'
        msg: "INSTALL_SCOPE must be 'user' or 'machine', got '{{.INSTALL_SCOPE}}'"
    cmds:
      # Create the Microsoft WebView2 bootstrapper if it doesn't exist
      - wails3 generate webview2bootstrapper -dir "{{.ROOT_DIR}}/build/windows/nsis"
      - |
        {{if eq OS "windows"}}
        makensis {{if eq .INSTALL_SCOPE "user"}}-DWAILS_INSTALL_SCOPE=user -DREQUEST_EXECUTION_LEVEL=user {{end}}-DARG_WAILS_{{.ARG_FLAG}}_BINARY="{{.ROOT_DIR}}\{{.BIN_DIR}}\{{.APP_NAME}}.exe" project.nsi
        {{else}}
        makensis {{if eq .INSTALL_SCOPE "user"}}-DWAILS_INSTALL_SCOPE=user -DREQUEST_EXECUTION_LEVEL=user {{end}}-DARG_WAILS_{{.ARG_FLAG}}_BINARY="{{.ROOT_DIR}}/{{.BIN_DIR}}/{{.APP_NAME}}.exe" project.nsi
        {{end}}
    vars:
      ARCH: '{{.ARCH | default ARCH}}'
      ARG_FLAG: '{{if eq .ARCH "amd64"}}AMD64{{else}}ARM64{{end}}'
      INSTALL_SCOPE: '{{.INSTALL_SCOPE | default "machine"}}'

  create:msix:package:
    summary: Creates an MSIX package
    deps:
      - task: build
    cmds:
      - |-
        wails3 tool msix \
          --config "{{.ROOT_DIR}}/build/config.yml" \
          --name "{{.APP_NAME}}.exe" \
          --executable "{{.ROOT_DIR}}/{{.BIN_DIR}}/{{.APP_NAME}}.exe" \
          --arch "{{.ARCH}}" \
          --out "{{.ROOT_DIR}}/{{.BIN_DIR}}/{{.APP_NAME}}-{{.ARCH}}.msix" \
          {{if .CERT_PATH}}--cert "{{.CERT_PATH}}"{{end}} \
          {{if .PUBLISHER}}--publisher "{{.PUBLISHER}}"{{end}} \
          {{if eq .USE_MSIX_TOOL "true"}}--use-msix-tool{{else}}--use-makeappx{{end}}
    vars:
      ARCH: '{{.ARCH | default ARCH}}'
      CERT_PATH: '{{.CERT_PATH | default ""}}'
      PUBLISHER: '{{.PUBLISHER | default ""}}'
      USE_MSIX_TOOL: '{{.USE_MSIX_TOOL | default "false"}}'

  install:msix:tools:
    summary: Installs tools required for MSIX packaging
    cmds:
      - wails3 tool msix-install-tools

  run:
    cmds:
      - '{{.BIN_DIR}}/{{.APP_NAME}}.exe'

  sign:
    summary: Signs the Windows executable
    desc: |
      Signs the .exe with an Authenticode certificate.
      Set SIGN_CERTIFICATE or SIGN_THUMBPRINT to override the certificate
      configured globally via `wails3 setup` (~/.config/wails/defaults.yaml).
      Password is retrieved from system keychain (run: wails3 setup signing)
    deps:
      - task: build
    cmds:
      # Certificate vars are optional: if unset, `wails3 tool sign` falls back to
      # the certificate configured globally via `wails3 setup`.
      - wails3 tool sign --input "{{.BIN_DIR}}/{{.APP_NAME}}.exe" {{if .SIGN_CERTIFICATE}}--certificate "{{.SIGN_CERTIFICATE}}"{{end}} {{if .SIGN_THUMBPRINT}}--thumbprint "{{.SIGN_THUMBPRINT}}"{{end}} {{if .TIMESTAMP_SERVER}}--timestamp "{{.TIMESTAMP_SERVER}}"{{end}}

  sign:installer:
    summary: Signs the NSIS installer
    desc: |
      Creates and signs the NSIS installer.
      Set SIGN_CERTIFICATE or SIGN_THUMBPRINT to override the certificate
      configured globally via `wails3 setup` (~/.config/wails/defaults.yaml).
      Password is retrieved from system keychain (run: wails3 setup signing)
    deps:
      - task: create:nsis:installer
        vars:
          INSTALL_SCOPE: '{{.INSTALL_SCOPE | default "machine"}}'
    vars:
      INSTALL_SCOPE: '{{.INSTALL_SCOPE | default "machine"}}'
    cmds:
      - wails3 tool sign --input "build/windows/nsis/{{.APP_NAME}}-installer.exe" {{if .SIGN_CERTIFICATE}}--certificate "{{.SIGN_CERTIFICATE}}"{{end}} {{if .SIGN_THUMBPRINT}}--thumbprint "{{.SIGN_THUMBPRINT}}"{{end}} {{if .TIMESTAMP_SERVER}}--timestamp "{{.TIMESTAMP_SERVER}}"{{end}}

========== build\windows\nsis\project.nsi ==========
Unicode true

####
## Please note: Template replacements don't work in this file. They are provided with default defines like
## mentioned underneath.
## If the keyword is not defined, "wails_tools.nsh" will populate them.
## If they are defined here, "wails_tools.nsh" will not touch them. This allows you to use this project.nsi manually
## from outside of Wails for debugging and development of the installer.
##
## For development first make a wails nsis build to populate the "wails_tools.nsh":
## > wails build --target windows/amd64 --nsis
## Then you can call makensis on this file with specifying the path to your binary:
## For a AMD64 only installer:
## > makensis -DARG_WAILS_AMD64_BINARY=..\..\bin\app.exe
## For a ARM64 only installer:
## > makensis -DARG_WAILS_ARM64_BINARY=..\..\bin\app.exe
## For a installer with both architectures:
## > makensis -DARG_WAILS_AMD64_BINARY=..\..\bin\app-amd64.exe -DARG_WAILS_ARM64_BINARY=..\..\bin\app-arm64.exe
####
## The following information is taken from the wails_tools.nsh file, but they can be overwritten here.
####
## !define INFO_PROJECTNAME    "my-project" # Default "NovaHost"
## !define INFO_COMPANYNAME    "My Company" # Default "NovaHost"
## !define INFO_PRODUCTNAME    "My Product Name" # Default "NovaHost Desktop"
## !define INFO_PRODUCTVERSION "1.0.0"     # Default "0.1.0"
## !define INFO_COPYRIGHT      "(c) Now, My Company" # Default "Â© 2026, My Company"
###
## !define PRODUCT_EXECUTABLE  "Application.exe"      # Default "${INFO_PROJECTNAME}.exe"
## !define UNINST_KEY_NAME     "UninstKeyInRegistry"  # Default "${INFO_COMPANYNAME}${INFO_PRODUCTNAME}"
####
## !define REQUEST_EXECUTION_LEVEL "admin"            # Default "admin"  see also https://nsis.sourceforge.io/Docs/Chapter4.html
## !define WAILS_INSTALL_SCOPE     "user"             # Default "machine" - set to "user" for per-user install ($LOCALAPPDATA) without UAC prompt
####
## Include the wails tools
####
!include "wails_tools.nsh"

# The version information for this two must consist of 4 parts
VIProductVersion "${INFO_PRODUCTVERSION}.0"
VIFileVersion    "${INFO_PRODUCTVERSION}.0"

VIAddVersionKey "CompanyName"     "${INFO_COMPANYNAME}"
VIAddVersionKey "FileDescription" "${INFO_PRODUCTNAME} Installer"
VIAddVersionKey "ProductVersion"  "${INFO_PRODUCTVERSION}"
VIAddVersionKey "FileVersion"     "${INFO_PRODUCTVERSION}"
VIAddVersionKey "LegalCopyright"  "${INFO_COPYRIGHT}"
VIAddVersionKey "ProductName"     "${INFO_PRODUCTNAME}"

# Enable HiDPI support. https://nsis.sourceforge.io/Reference/ManifestDPIAware
ManifestDPIAware true

!include "MUI.nsh"

!define MUI_ICON "..\icon.ico"
!define MUI_UNICON "..\icon.ico"
# !define MUI_WELCOMEFINISHPAGE_BITMAP "resources\leftimage.bmp" #Include this to add a bitmap on the left side of the Welcome Page. Must be a size of 164x314
!define MUI_FINISHPAGE_NOAUTOCLOSE # Wait on the INSTFILES page so the user can take a look into the details of the installation steps
!define MUI_ABORTWARNING # This will warn the user if they exit from the installer.

!insertmacro MUI_PAGE_WELCOME # Welcome to the installer page.
# !insertmacro MUI_PAGE_LICENSE "resources\eula.txt" # Adds a EULA page to the installer
!insertmacro MUI_PAGE_DIRECTORY # In which folder install page.
!insertmacro MUI_PAGE_INSTFILES # Installing page.
!insertmacro MUI_PAGE_FINISH # Finished installation page.

!insertmacro MUI_UNPAGE_INSTFILES # Uninstalling page

!insertmacro MUI_LANGUAGE "English" # Set the Language of the installer

## The following two statements can be used to sign the installer and the uninstaller. The path to the binaries are provided in %1
#!uninstfinalize 'signtool --file "%1"'
#!finalize 'signtool --file "%1"'

Name "${INFO_PRODUCTNAME}"
OutFile "..\..\..\bin\${INFO_PROJECTNAME}-${ARCH}-installer.exe" # Name of the installer's file.
!if "${WAILS_INSTALL_SCOPE}" == "user"
    InstallDir "$LOCALAPPDATA\Programs\${INFO_PRODUCTNAME}"
!else
    InstallDir "$PROGRAMFILES64\${INFO_COMPANYNAME}\${INFO_PRODUCTNAME}"
!endif
ShowInstDetails show # This will always show the installation details.

Function .onInit
   !insertmacro wails.checkArchitecture
FunctionEnd

Section
    !insertmacro wails.setShellContext

    !insertmacro wails.webview2runtime

    SetOutPath $INSTDIR

    !insertmacro wails.files

    CreateShortcut "$SMPROGRAMS\${INFO_PRODUCTNAME}.lnk" "$INSTDIR\${PRODUCT_EXECUTABLE}"
    CreateShortCut "$DESKTOP\${INFO_PRODUCTNAME}.lnk" "$INSTDIR\${PRODUCT_EXECUTABLE}"

    !insertmacro wails.associateFiles
    !insertmacro wails.associateCustomProtocols

    !insertmacro wails.writeUninstaller
SectionEnd

Section "uninstall"
    !insertmacro wails.setShellContext

    RMDir /r "$AppData\${PRODUCT_EXECUTABLE}" # Remove the WebView2 DataPath

    RMDir /r $INSTDIR

    Delete "$SMPROGRAMS\${INFO_PRODUCTNAME}.lnk"
    Delete "$DESKTOP\${INFO_PRODUCTNAME}.lnk"

    !insertmacro wails.unassociateFiles
    !insertmacro wails.unassociateCustomProtocols

    !insertmacro wails.deleteUninstaller
SectionEnd
