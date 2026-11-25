===== SERVICES =====


===== PROJECT CONFIGURATION =====

===== MAIN WINDOW CODE =====

===== SERVICE FILES =====

--- CoreDiscoveryService.cs ---

--- LogMonitorService.cs ---

--- ServerStatusService.cs ---
FullName
--------
D:\NovaHost\NovaHost.Desktop\Services\CoreDiscoveryService.cs
D:\NovaHost\NovaHost.Desktop\Services\CoreDiscoveryService.cs.20261101-202456.bak
D:\NovaHost\NovaHost.Desktop\Services\LogMonitorService.cs
D:\NovaHost\NovaHost.Desktop\Services\ServerStatusService.cs
D:\NovaHost\NovaHost.Desktop\Services\ServerStatusService.cs.20261101-192010.bak
D:\NovaHost\NovaHost.Desktop\Services\ServerStatusService.cs.20261101-202929.bak
D:\NovaHost\NovaHost.Desktop\Services\ServerStatusService.cs.20261102-153423.bak
D:\NovaHost\NovaHost.Desktop\Services\ServerStatusService.cs.20261102-154811.bak
D:\NovaHost\NovaHost.Desktop\Services\ServerStatusService.cs.before-detection-fix-20261101-192759.bak
<Project Sdk="Microsoft.NET.Sdk">

  <PropertyGroup>
    <OutputType>WinExe</OutputType>
    <TargetFramework>net10.0-windows</TargetFramework>
    <Nullable>enable</Nullable>
    <ImplicitUsings>enable</ImplicitUsings>
    <UseWPF>true</UseWPF>
    <ApplicationIcon>Assets\novahost.ico</ApplicationIcon>
  </PropertyGroup>

  <ItemGroup>
    <PackageReference Include="Microsoft.Web.WebView2" Version="1.0.4314.35" />
  </ItemGroup>

  <ItemGroup>
    <None Update="wwwroot\**\*" CopyToOutputDirectory="PreserveNewest" />
    <None Remove="Assets\novahost.ico" />
    <Resource Include="Assets\novahost.ico" />
  </ItemGroup>

</Project>

using System;
using System.IO;
using System.Net.Http;
using System.Text.Json;
using System.Threading;
using System.Threading.Tasks;
using System.Windows;
using Microsoft.Web.WebView2.Core;
using NovaHost.Desktop.Services;

namespace NovaHost.Desktop;

public partial class MainWindow : Window
{
    private static readonly HttpClient Http = new()
    {
        Timeout = TimeSpan.FromSeconds(12)
    };

    private LogMonitorService? _logMonitor;
    private bool _logMonitorStarted;
    private CoreDiscoverySnapshot? _coreDiscovery;
    private string? _novaHostRoot;

    public MainWindow()
    {
        InitializeComponent();
        Loaded += MainWindow_Loaded;
        Closed += MainWindow_Closed;
    }

    private async void MainWindow_Loaded(object sender, RoutedEventArgs e)
    {
        try
        {
            await NovaHostView.EnsureCoreWebView2Async();

            var core = NovaHostView.CoreWebView2;

            // Discover the NovaHost installation.
            var discovery = CoreDiscoveryService.Discover();
            _novaHostRoot = discovery.NovaHostRoot;
            _coreDiscovery = discovery;

            core.Settings.AreDefaultContextMenusEnabled = false;
            core.Settings.IsStatusBarEnabled = false;

            core.WebMessageReceived += CoreWebView2_WebMessageReceived;
            core.NavigationCompleted += NovaHostView_NavigationCompleted;

            string htmlPath = Path.Combine(
                AppContext.BaseDirectory,
                "wwwroot",
                "index.html"
            );

            if (!File.Exists(htmlPath))
            {
                MessageBox.Show(
                    $"Frontend file not found:\n{htmlPath}",
                    "NovaHost",
                    MessageBoxButton.OK,
                    MessageBoxImage.Error
                );
                return;
            }

            NovaHostView.Source = new Uri(htmlPath);
        }
        catch (Exception ex)
        {
            MessageBox.Show(
                "WebView2 initialization failed.\n\n" + ex.Message,
                "NovaHost",
                MessageBoxButton.OK,
                MessageBoxImage.Error
            );
        }
    }

    private async void CoreWebView2_WebMessageReceived(
        object? sender,
        CoreWebView2WebMessageReceivedEventArgs e)
    {
        try
        {
            using JsonDocument document =
                JsonDocument.Parse(e.WebMessageAsJson);

            JsonElement root = document.RootElement;

            if (!root.TryGetProperty("type", out JsonElement typeElement))
                return;

            string type = typeElement.GetString() ?? "";

            string requestId = root.TryGetProperty("requestId", out var id)
                ? id.GetString() ?? ""
                : "";

            switch (type)
            {
                case "citySearch":
                {
                    string query = root.TryGetProperty("query", out var q)
                        ? q.GetString()?.Trim() ?? ""
                        : "";

                    await SearchCitiesAsync(query, requestId);
                    break;
                }

                case "networkRefresh":
                {
                    await RefreshNetworkAsync(requestId);
                    break;
                }

                case "serverStatusRefresh":
                {
                    RefreshServerStatus(requestId);
                    break;
                }
            }
        }
        catch (Exception ex)
        {
            System.Diagnostics.Debug.WriteLine(
                "WebView2 message error: " + ex.Message
            );
        }
    }

    private void RefreshServerStatus(string requestId)
    {
        try
        {
            if (string.IsNullOrWhiteSpace(_novaHostRoot))
                throw new InvalidOperationException(
                    "NovaHost installation was not found."
                );

            var components =
                ServerStatusService.GetSnapshot(
                    _novaHostRoot,
                    _coreDiscovery?.Components);

            SendToFrontend(new
            {
                type = "serverStatusResult",
                requestId,
                components
            });
        }
        catch (Exception ex)
        {
            SendToFrontend(new
            {
                type = "serverStatusError",
                requestId,
                error = ex.Message
            });
        }
    }

    private void NovaHostView_NavigationCompleted(
        object? sender,
        CoreWebView2NavigationCompletedEventArgs e)
    {
        if (!e.IsSuccess)
        {
            SendToFrontend(new
            {
                type = "logMonitorStatus",
                status = "offline",
                message = "Frontend navigation failed."
            });

            return;
        }

        if (_logMonitorStarted)
            return;

        try
        {
            if (string.IsNullOrWhiteSpace(_novaHostRoot))
                throw new InvalidOperationException(
                    "NovaHost installation was not found. Log monitoring cannot start."
                );

            _logMonitor = new LogMonitorService(_novaHostRoot);

            _logMonitor.EntryReceived += OnLogEntryReceived;

            _logMonitorStarted = true;
            _logMonitor.Start();

            SendToFrontend(new
            {
                type = "logMonitorStatus",
                status = "live",
                message = "Log monitoring is active."
            });
        }
        catch (Exception ex)
        {
            _logMonitorStarted = false;

            if (_logMonitor != null)
            {
                _logMonitor.EntryReceived -= OnLogEntryReceived;
                _logMonitor.Dispose();
                _logMonitor = null;
            }

            SendToFrontend(new
            {
                type = "logMonitorStatus",
                status = "offline",
                message = ex.Message
            });
        }
    }

    private void OnLogEntryReceived(LogEntry entry)
    {
        if (Dispatcher.HasShutdownStarted ||
            Dispatcher.HasShutdownFinished)
        {
            return;
        }

        Dispatcher.BeginInvoke(new Action(() =>
        {
            SendToFrontend(new
            {
                type = "logEntry",
                entry = new
                {
                    timestamp = entry.Timestamp.ToString("O"),
                    module = entry.Module,
                    level = entry.Level,
                    message = entry.Message,
                    source = entry.Source
                }
            });
        }));
    }

    private void MainWindow_Closed(object? sender, EventArgs e)
    {
        if (_logMonitor != null)
        {
            _logMonitor.EntryReceived -= OnLogEntryReceived;
            _logMonitor.Dispose();
            _logMonitor = null;
        }

        _logMonitorStarted = false;
    }

    private async Task SearchCitiesAsync(
        string query,
        string requestId)
    {
        try
        {
            if (query.Length < 2 || query.Length > 100)
                throw new ArgumentException("Invalid city search query.");

            string url =
                "https://geocoding-api.open-meteo.com/v1/search" +
                "?name=" + Uri.EscapeDataString(query) +
                "&count=20&language=en&format=json";

            using var response = await Http.GetAsync(url);
            response.EnsureSuccessStatusCode();

            string json = await response.Content.ReadAsStringAsync();

            using JsonDocument result = JsonDocument.Parse(json);

            SendToFrontend(new
            {
                type = "citySearchResult",
                requestId,
                data = result.RootElement.Clone()
            });
        }
        catch (Exception ex)
        {
            SendToFrontend(new
            {
                type = "citySearchError",
                requestId,
                error = ex.Message
            });
        }
    }

    private async Task RefreshNetworkAsync(string requestId)
    {
        string? ipv4 = null;
        string? ipv6 = null;
        JsonElement? geo = null;

        try
        {
            var ipv4Task = GetPublicIpAsync(
                "https://api.ipify.org?format=json"
            );

            var ipv6Task = GetPublicIpAsync(
                "https://api6.ipify.org?format=json"
            );

            await Task.WhenAll(ipv4Task, ipv6Task);

            ipv4 = ipv4Task.Result;
            ipv6 = ipv6Task.Result;

            string? address = ipv4 ?? ipv6;

            if (!string.IsNullOrWhiteSpace(address))
            {
                string geoUrl =
                    "https://ipwho.is/" + Uri.EscapeDataString(address);

                using var response = await Http.GetAsync(geoUrl);
                response.EnsureSuccessStatusCode();

                using JsonDocument geoDocument =
                    JsonDocument.Parse(
                        await response.Content.ReadAsStringAsync()
                    );

                geo = geoDocument.RootElement.Clone();
            }

            SendToFrontend(new
            {
                type = "networkResult",
                requestId,
                ipv4,
                ipv6,
                geo
            });
        }
        catch (Exception ex)
        {
            SendToFrontend(new
            {
                type = "networkError",
                requestId,
                ipv4,
                ipv6,
                error = ex.Message
            });
        }
    }

    private static async Task<string?> GetPublicIpAsync(string url)
    {
        try
        {
            using var timeout = new CancellationTokenSource(
                TimeSpan.FromSeconds(6)
            );

            string json = await Http.GetStringAsync(url, timeout.Token);

            using JsonDocument document = JsonDocument.Parse(json);

            if (document.RootElement.TryGetProperty(
                "ip", out JsonElement ipElement))
            {
                return ipElement.GetString();
            }
        }
        catch
        {
            // IPv4 or IPv6 may not be available.
        }

        return null;
    }

    private void SendToFrontend(object message)
    {
        try
        {
            if (NovaHostView.CoreWebView2 == null)
                return;

            string json = JsonSerializer.Serialize(message);

            NovaHostView.CoreWebView2.PostWebMessageAsJson(json);
        }
        catch (Exception ex)
        {
            System.Diagnostics.Debug.WriteLine(
                "Frontend message error: " + ex.Message
            );
        }
    }
}

using System;
using System.Collections.Generic;
using System.Diagnostics;
using System.IO;
using System.Linq;

namespace NovaHost.Desktop.Services;

public sealed record DiscoveredComponent
{
    public string Name { get; init; } = "";
    public string Category { get; init; } = "";
    public string Path { get; init; } = "";
    public bool Exists { get; init; }
}

public sealed record CoreDiscoverySnapshot
{
    public string? NovaHostRoot { get; init; }

    public IReadOnlyList<DiscoveredComponent> Components { get; init; }
        = Array.Empty<DiscoveredComponent>();
}

public static class CoreDiscoveryService
{
    public static CoreDiscoverySnapshot Discover()
    {
        string? root = DiscoverNovaHostRoot();

        var components = new List<DiscoveredComponent>
        {
            Create("Apache", "Server", root,
                @"Runtime\Apache\Apache24\bin\httpd.exe"),

            Create("MariaDB", "Database", root,
                @"Runtime\MariaDB\bin\mysqld.exe"),

            Create("PHP", "Runtime", root,
                @"Runtime\PHP\8.3.36\php.exe"),

            Create("Nginx", "Server", root,
                @"Runtime\Nginx\nginx-1.30.6\nginx.exe"),

            Create("Redis", "Cache", root,
                @"Runtime\Memurai\memurai.exe"),

            Create("FileZilla", "Server", root,
                @"Runtime\FileZilla\FileZilla server.exe"),

            Create("Mercury", "Server", root,
                @"Runtime\Mercury\mercury.exe"),

            Create("Node.js", "Runtime", null, null,
                "node.exe"),

            CreatePython()
        };

        return new CoreDiscoverySnapshot
        {
            NovaHostRoot = root,
            Components = components
        };
    }

    private static string? DiscoverNovaHostRoot()
    {
        var candidates = new List<string>();

        string? environmentRoot =
            Environment.GetEnvironmentVariable("NOVAHOST_ROOT");

        if (!string.IsNullOrWhiteSpace(environmentRoot))
            candidates.Add(environmentRoot);

        foreach (string start in new[]
        {
            AppContext.BaseDirectory,
            Environment.CurrentDirectory
        })
        {
            DirectoryInfo? directory = new(start);

            while (directory != null)
            {
                candidates.Add(directory.FullName);
                directory = directory.Parent;
            }
        }

        foreach (string candidate in candidates.Distinct(
            StringComparer.OrdinalIgnoreCase))
        {
            try
            {
                string fullPath = Path.GetFullPath(candidate);

                if (IsNovaHostRoot(fullPath))
                    return fullPath;
            }
            catch
            {
                // Ignore invalid or inaccessible paths.
            }
        }

        return null;
    }

    private static bool IsNovaHostRoot(string root)
    {
        return
            File.Exists(Path.Combine(
                root, "Runtime", "Apache", "Apache24",
                "bin", "httpd.exe")) &&
            File.Exists(Path.Combine(
                root, "Runtime", "MariaDB",
                "bin", "mysqld.exe"));
    }

    private static DiscoveredComponent Create(
        string name,
        string category,
        string? root,
        string? relativePath,
        string? executableName = null)
    {
        string path = "";

        if (!string.IsNullOrWhiteSpace(root) &&
            !string.IsNullOrWhiteSpace(relativePath))
        {
            string candidate = Path.Combine(root, relativePath);

            if (File.Exists(candidate))
                path = candidate;
        }

        if (string.IsNullOrWhiteSpace(path) &&
            !string.IsNullOrWhiteSpace(executableName))
        {
            path = FindOnPath(executableName) ?? "";
        }

        return new DiscoveredComponent
        {
            Name = name,
            Category = category,
            Path = path,
            Exists = !string.IsNullOrWhiteSpace(path)
        };
    }

    private static DiscoveredComponent CreatePython()
    {
        string? path = FindOnPath("python.exe");

        if (string.IsNullOrWhiteSpace(path))
        {
            string profile =
                Environment.GetFolderPath(
                    Environment.SpecialFolder.UserProfile);

            if (!string.IsNullOrWhiteSpace(profile))
            {
                string pythonRoot = Path.Combine(
                    profile,
                    "AppData", "Local", "Programs", "Python");

                try
                {
                    if (Directory.Exists(pythonRoot))
                    {
                        path = Directory.GetDirectories(
                                pythonRoot, "Python*")
                            .OrderByDescending(x => x)
                            .Select(x => Path.Combine(x, "python.exe"))
                            .FirstOrDefault(File.Exists);
                    }
                }
                catch
                {
                    // Ignore inaccessible directories.
                }
            }
        }

        return new DiscoveredComponent
        {
            Name = "Python",
            Category = "Runtime",
            Path = path ?? "",
            Exists = !string.IsNullOrWhiteSpace(path)
        };
    }

    private static string? FindOnPath(string executableName)
    {
        string? pathValue =
            Environment.GetEnvironmentVariable("PATH");

        if (string.IsNullOrWhiteSpace(pathValue))
            return null;

        foreach (string directory in pathValue.Split(
            Path.PathSeparator,
            StringSplitOptions.RemoveEmptyEntries))
        {
            try
            {
                string candidate = Path.Combine(
                    directory.Trim().Trim('"'),
                    executableName);

                if (File.Exists(candidate))
                    return Path.GetFullPath(candidate);
            }
            catch
            {
                // Ignore invalid PATH entries.
            }
        }

        return null;
    }
}

using System;
using System.Collections.Generic;
using System.Globalization;
using System.IO;
using System.Linq;
using System.Text;
using System.Threading;

namespace NovaHost.Desktop.Services;

public sealed record LogEntry
{
    public DateTimeOffset Timestamp { get; init; }
    public string Module { get; init; } = "";
    public string Level { get; init; } = "";
    public string Message { get; init; } = "";
    public string Source { get; init; } = "";
}

public sealed class LogMonitorService : IDisposable
{
    private readonly string[] _files;
    private readonly Dictionary<string, long> _positions = new();
    private readonly Dictionary<string, string> _pending = new();
    private readonly object _sync = new();

    private Timer? _timer;
    private bool _started;
    private bool _disposed;
    private int _polling;

    public event Action<LogEntry>? EntryReceived;

    public LogMonitorService(string novaHostRoot)
    {
        string root = Path.GetFullPath(novaHostRoot);

        var files = new List<string>
        {
            Path.Combine(
                root, "Runtime", "Apache", "Apache24",
                "logs", "error_log"),

            Path.Combine(
                root, "Runtime", "Nginx",
                "nginx-1.30.6", "logs", "error.log")
        };

        string mariaDbData = Path.Combine(
            root, "Runtime", "MariaDB", "data");

        try
        {
            if (Directory.Exists(mariaDbData))
            {
                files.AddRange(
                    Directory.GetFiles(
                        mariaDbData, "*.err",
                        SearchOption.TopDirectoryOnly));
            }
        }
        catch (UnauthorizedAccessException ex)
        {
            System.Diagnostics.Debug.WriteLine(
                "MariaDB log discovery failed: " + ex.Message);
        }
        catch (IOException ex)
        {
            System.Diagnostics.Debug.WriteLine(
                "MariaDB log discovery failed: " + ex.Message);
        }

        _files = files
            .Distinct(StringComparer.OrdinalIgnoreCase)
            .ToArray();
    }

    public void Start()
    {
        lock (_sync)
        {
            if (_started || _disposed)
                return;

            _started = true;

            foreach (string path in _files)
            {
                InitializeFile(path);
            }

            _timer = new Timer(
                _ => Poll(),
                null,
                TimeSpan.FromSeconds(1),
                TimeSpan.FromSeconds(1));
        }
    }

    private void InitializeFile(string path)
    {
        try
        {
            if (!File.Exists(path))
                return;

            string[] lines = File.ReadLines(path)
                .Where(line => !string.IsNullOrWhiteSpace(line))
                .TakeLast(30)
                .ToArray();

            DateTimeOffset fallback =
                new(File.GetLastWriteTime(path));

            foreach (string line in lines)
            {
                Publish(path, line, fallback);
            }

            _positions[path] = new FileInfo(path).Length;
            _pending[path] = "";
        }
        catch (Exception ex)
        {
            System.Diagnostics.Debug.WriteLine(
                "Log initialization failed: " + ex.Message);
        }
    }

    private void Poll()
    {
        if (Interlocked.Exchange(ref _polling, 1) != 0)
            return;

        try
        {
            lock (_sync)
            {
                if (_disposed)
                    return;

                foreach (string path in _files)
                {
                    ReadNewEntries(path);
                }
            }
        }
        catch (Exception ex)
        {
            System.Diagnostics.Debug.WriteLine(
                "Log polling failed: " + ex.Message);
        }
        finally
        {
            Interlocked.Exchange(ref _polling, 0);
        }
    }

    private void ReadNewEntries(string path)
    {
        try
        {
            if (!File.Exists(path))
                return;

            if (!_positions.ContainsKey(path))
            {
                InitializeFile(path);
                return;
            }

            using var stream = new FileStream(
                path,
                FileMode.Open,
                FileAccess.Read,
                FileShare.ReadWrite | FileShare.Delete);

            if (stream.Length < _positions[path])
            {
                _positions[path] = 0;
                _pending[path] = "";
            }

            if (stream.Length == _positions[path])
                return;

            stream.Position = _positions[path];

            using var reader = new StreamReader(
                stream,
                Encoding.UTF8,
                true,
                4096,
                true);

            string chunk = reader.ReadToEnd();
            _positions[path] = stream.Position;

            string combined =
                (_pending.TryGetValue(path, out string? pending)
                    ? pending
                    : "") + chunk;

            string[] lines = combined.Split('\n');

            _pending[path] = lines[^1];

            for (int i = 0; i < lines.Length - 1; i++)
            {
                string line = lines[i].TrimEnd('\r');

                if (!string.IsNullOrWhiteSpace(line))
                {
                    Publish(
                        path,
                        line,
                        new DateTimeOffset(
                            File.GetLastWriteTime(path)));
                }
            }
        }
        catch (IOException)
        {
            // The log file may be temporarily locked or rotated.
        }
        catch (UnauthorizedAccessException ex)
        {
            System.Diagnostics.Debug.WriteLine(
                "Log access denied: " + ex.Message);
        }
    }

    private void Publish(
        string path,
        string line,
        DateTimeOffset fallback)
    {
        string fileName = Path.GetFileName(path);
        string fullPath = path.ToLowerInvariant();

        string module =
            fullPath.Contains(
                Path.Combine("runtime", "mariadb"),
                StringComparison.OrdinalIgnoreCase)
                ? "database"
                : fullPath.Contains(
                    Path.Combine("runtime", "nginx"),
                    StringComparison.OrdinalIgnoreCase)
                    ? "nginx"
                    : line.Contains(
                        "[php:",
                        StringComparison.OrdinalIgnoreCase)
                        ? "php"
                        : "apache";

        string lower = line.ToLowerInvariant();

        string level =
            lower.Contains("[error]") ||
            lower.Contains("fatal error") ||
            lower.Contains("exception") ||
            lower.Contains("error")
                ? "error"
                : lower.Contains("[warn]") ||
                  lower.Contains("warning")
                    ? "warning"
                    : lower.Contains("ready for connections") ||
                      lower.Contains("shutdown complete")
                        ? "success"
                        : "info";

        EntryReceived?.Invoke(new LogEntry
        {
            Timestamp = ParseTimestamp(line, fallback),
            Module = module,
            Level = level,
            Message = line,
            Source = fileName
        });
    }

    private static DateTimeOffset ParseTimestamp(
        string line,
        DateTimeOffset fallback)
    {
        if (line.StartsWith("[", StringComparison.Ordinal))
        {
            int end = line.IndexOf(']');

            if (end > 1)
            {
                string value = line.Substring(1, end - 1);

                if (DateTime.TryParseExact(
                    value,
                    "ddd MMM dd HH:mm:ss.ffffff yyyy",
                    CultureInfo.InvariantCulture,
                    DateTimeStyles.AssumeLocal,
                    out DateTime apacheTime))
                {
                    return new DateTimeOffset(apacheTime);
                }
            }
        }

        if (line.Length >= 19 &&
            DateTime.TryParseExact(
                line.Substring(0, 19),
                "yyyy-MM-dd HH:mm:ss",
                CultureInfo.InvariantCulture,
                DateTimeStyles.AssumeLocal,
                out DateTime databaseTime))
        {
            return new DateTimeOffset(databaseTime);
        }

        return fallback;
    }

    public void Dispose()
    {
        lock (_sync)
        {
            if (_disposed)
                return;

            _disposed = true;
            _timer?.Dispose();
            _timer = null;
        }
    }
}

using System;
using System.Collections.Generic;
using System.Diagnostics;
using System.IO;
using System.Linq;
using System.Net.NetworkInformation;

namespace NovaHost.Desktop.Services;

public sealed class ServerComponentStatus
{
    public string Component { get; init; } = "";
    public string Status { get; init; } = "";
    public string Detail { get; init; } = "";
    public string ExecutablePath { get; init; } = "";
    public int? ProcessId { get; init; }
    public int? Port { get; init; }
    public string Version { get; init; } = "";
}

public static class ServerStatusService
{
    public static IReadOnlyList<ServerComponentStatus> GetSnapshot(
        string novaHostRoot,
        IReadOnlyList<DiscoveredComponent>? discoveredComponents = null)
    {
        string root = Path.GetFullPath(novaHostRoot);

        string apachePath = GetPath(
            discoveredComponents, "Apache",
            Path.Combine(root, "Runtime", "Apache",
                "Apache24", "bin", "httpd.exe"));

        string mariaDbPath = GetPath(
            discoveredComponents, "MariaDB",
            Path.Combine(root, "Runtime", "MariaDB",
                "bin", "mysqld.exe"));

        string phpPath = GetPath(
            discoveredComponents, "PHP",
            Path.Combine(root, "Runtime", "PHP",
                "8.3.36", "php.exe"));

        string nginxPath = GetPath(
            discoveredComponents, "Nginx",
            Path.Combine(root, "Runtime", "Nginx",
                "nginx-1.30.6", "nginx.exe"));

        string memuraiPath = GetPath(
            discoveredComponents, "Redis",
            Path.Combine(root, "Runtime", "Memurai",
                "memurai.exe"));

        string fileZillaPath = GetPath(
            discoveredComponents, "FileZilla",
            Path.Combine(root, "Runtime", "FileZilla",
                "FileZilla server.exe"));

        string mercuryPath = GetPath(
            discoveredComponents, "Mercury",
            Path.Combine(root, "Runtime", "Mercury",
                "mercury.exe"));

        string nodePath = GetPath(
            discoveredComponents, "Node.js", "");

        string pythonPath = GetPath(
            discoveredComponents, "Python", "");

                string mariaDbConfigPath = Path.Combine(
            root, "Runtime", "MariaDB", "data", "my.ini");

        int? mariaDbPort = ReadMariaDbPort(mariaDbConfigPath);

        ServerComponentStatus mariaDbStatus = mariaDbPort.HasValue
            ? CheckWindowsService(
                "MariaDB", "NovaHostDB", mariaDbPath, mariaDbPort)
            : new ServerComponentStatus
            {
                Component = "MariaDB",
                Status = "WARNING",
                Detail = "MariaDB port could not be read from my.ini [mysqld].",
                ExecutablePath = mariaDbPath
            };

        return new[]
        {
            CheckServer("Apache",
                apachePath, "httpd", 8082),

            mariaDbStatus,

            CheckServer("Nginx",
                nginxPath, "nginx", null),

            CheckServer("Redis",
                memuraiPath, "memurai", 6379),

            CheckWindowsService("FileZilla",
                "FileZilla Server", fileZillaPath),

            CheckServer("Mercury",
                mercuryPath, "mercury", null),

            CheckRuntime("PHP", phpPath, "-v"),

            CheckRuntime("Node.js", nodePath, "--version"),

            CheckRuntime("Python", pythonPath, "--version")
        };
    }

    private static string GetPath(
        IReadOnlyList<DiscoveredComponent>? components,
        string name,
        string fallbackPath)
    {
        if (components != null)
        {
            foreach (DiscoveredComponent component in components)
            {
                if (string.Equals(
                    component.Name,
                    name,
                    StringComparison.OrdinalIgnoreCase) &&
                    component.Exists &&
                    !string.IsNullOrWhiteSpace(component.Path))
                {
                    return component.Path;
                }
            }
        }

        return fallbackPath;
    }

    private static ServerComponentStatus CheckServer(
        string component,
        string executablePath,
        string processName,
        int? port)
    {
        bool exists = File.Exists(executablePath);
        Process? process = FindProcess(executablePath, processName);

        bool portListening = port.HasValue &&
            IsPortListening(port.Value);

        string status;
        string detail;

        if (process != null && (!port.HasValue || portListening))
        {
            status = "RUNNING";
            detail = "Process detected";
        }
        else if (process != null)
        {
            status = "PROCESS DETECTED";
            detail = $"Process detected, but port {port} is not listening";
        }
        else if (exists && portListening)
        {
            status = "PORT CONFLICT";
            detail = $"Port {port} is used by another process";
        }
        else if (exists)
        {
            status = "STOPPED";
            detail = "Executable found, process not running";
        }
        else
        {
            status = "NOT FOUND";
            detail = "Executable not found";
        }

        int? processId = process?.Id;
        process?.Dispose();

        return new ServerComponentStatus
        {
            Component = component,
            Status = status,
            Detail = detail,
            ExecutablePath = executablePath,
            ProcessId = processId,
            Port = port
        };
    }

    private static int? ReadMariaDbPort(string configPath)
    {
        if (!File.Exists(configPath))
            return null;

        bool inMySqlSection = false;

        foreach (string rawLine in File.ReadLines(configPath))
        {
            string line = rawLine.Trim();

            if (string.IsNullOrWhiteSpace(line))
                continue;

            if (line.StartsWith("#") || line.StartsWith(";"))
                continue;

            if (line.StartsWith("[") && line.EndsWith("]"))
            {
                inMySqlSection = string.Equals(
                    line[1..^1].Trim(),
                    "mysqld",
                    StringComparison.OrdinalIgnoreCase);

                continue;
            }

            if (!inMySqlSection)
                continue;

            int separator = line.IndexOf('=');

            if (separator <= 0)
                continue;

            string key = line[..separator].Trim();
            string value = line[(separator + 1)..].Trim();

            if (!string.Equals(
                key, "port", StringComparison.OrdinalIgnoreCase))
                continue;

            if (int.TryParse(value, out int port) &&
                port >= 1 && port <= 65535)
            {
                return port;
            }

            return null;
        }

        return null;
    }
    private static ServerComponentStatus CheckWindowsService(
        string component,
        string serviceName,
        string executablePath,
        int? port = null)
    {
        try
        {
            using Process process = new();

            process.StartInfo = new ProcessStartInfo
            {
                FileName = Path.Combine(
                    Environment.SystemDirectory, "sc.exe"),
                Arguments = $"queryex \"{serviceName}\"",
                UseShellExecute = false,
                RedirectStandardOutput = true,
                RedirectStandardError = true,
                CreateNoWindow = true
            };

            process.Start();

            string output = process.StandardOutput.ReadToEnd();
            process.WaitForExit(5000);

            if (!process.HasExited)
            {
                process.Kill();

                return new ServerComponentStatus
                {
                    Component = component,
                    Status = "WARNING",
                    Detail = "Service query timed out",
                    ExecutablePath = executablePath,
                    Port = port
                };
            }

            if (process.ExitCode != 0)
            {
                bool listening = port.HasValue &&
                    IsPortListening(port.Value);

                return new ServerComponentStatus
                {
                    Component = component,
                    Status = listening ? "PORT CONFLICT" :
                        File.Exists(executablePath) ? "STOPPED" : "NOT FOUND",
                    Detail = $"Windows service {serviceName} was not found",
                    ExecutablePath = executablePath,
                    Port = port
                };
            }

            bool running = output.Contains(
                "RUNNING", StringComparison.OrdinalIgnoreCase);

            int? processId = null;

            foreach (string line in output.Split(
                new[] { '\r', '\n' },
                StringSplitOptions.RemoveEmptyEntries))
            {
                string trimmed = line.Trim();

                if (trimmed.StartsWith(
                    "PID", StringComparison.OrdinalIgnoreCase))
                {
                    string[] parts = trimmed.Split(':');

                    if (parts.Length > 1 &&
                        int.TryParse(parts[1].Trim(), out int pid) &&
                        pid > 0)
                    {
                        processId = pid;
                    }
                }
            }

            bool portListening = port.HasValue &&
                IsPortListening(port.Value);

            string status;
            string detail;

            if (running && (!port.HasValue || portListening))
            {
                status = "RUNNING";
                detail = "Windows service is running";
            }
            else if (running)
            {
                status = "PROCESS DETECTED";
                detail = $"Service is running, but port {port} is not listening";
            }
            else if (portListening)
            {
                status = "PORT CONFLICT";
                detail = $"Port {port} is used by another process";
            }
            else
            {
                status = "STOPPED";
                detail = "Windows service is stopped";
            }

            return new ServerComponentStatus
            {
                Component = component,
                Status = status,
                Detail = detail,
                ExecutablePath = executablePath,
                ProcessId = processId,
                Port = port
            };
        }
        catch (Exception ex)
        {
            return new ServerComponentStatus
            {
                Component = component,
                Status = "WARNING",
                Detail = ex.Message,
                ExecutablePath = executablePath,
                Port = port
            };
        }
    }

    private static ServerComponentStatus CheckRuntime(
        string component,
        string executablePath,
        string arguments)
    {
        if (!File.Exists(executablePath))
        {
            return new ServerComponentStatus
            {
                Component = component,
                Status = "NOT FOUND",
                Detail = "Runtime executable not found",
                ExecutablePath = executablePath
            };
        }

        try
        {
            using Process process = new();

            process.StartInfo = new ProcessStartInfo
            {
                FileName = executablePath,
                Arguments = arguments,
                UseShellExecute = false,
                RedirectStandardOutput = true,
                RedirectStandardError = true,
                CreateNoWindow = true
            };

            process.Start();

            string output = process.StandardOutput.ReadToEnd();
            string errorOutput = process.StandardError.ReadToEnd();
            process.WaitForExit(5000);

            if (!process.HasExited)
            {
                process.Kill();

                return new ServerComponentStatus
                {
                    Component = component,
                    Status = "AVAILABLE",
                    Detail = "Runtime executable found",
                    ExecutablePath = executablePath
                };
            }

            string version = (
                string.IsNullOrWhiteSpace(output) ? errorOutput : output)
                .Split(new[] { '\r', '\n' },
                    StringSplitOptions.RemoveEmptyEntries)
                .FirstOrDefault() ?? "";

            return new ServerComponentStatus
            {
                Component = component,
                Status = process.ExitCode == 0
                    ? "AVAILABLE" : "WARNING",
                Detail = version,
                ExecutablePath = executablePath,
                Version = version
            };
        }
        catch (Exception ex)
        {
            return new ServerComponentStatus
            {
                Component = component,
                Status = "WARNING",
                Detail = ex.Message,
                ExecutablePath = executablePath
            };
        }
    }

    private static Process? FindProcess(
        string executablePath,
        string processName)
    {
        if (!File.Exists(executablePath))
            return null;

        string expectedPath = Path.GetFullPath(executablePath);

        foreach (Process process in
            Process.GetProcessesByName(processName))
        {
            try
            {
                string? actualPath = process.MainModule?.FileName;

                if (!string.IsNullOrWhiteSpace(actualPath) &&
                    string.Equals(
                        Path.GetFullPath(actualPath),
                        expectedPath,
                        StringComparison.OrdinalIgnoreCase))
                {
                    return process;
                }
            }
            catch
            {
                // Windows may restrict process path access.
            }

            process.Dispose();
        }

        return null;
    }

    private static bool IsPortListening(int port)
    {
        try
        {
            return IPGlobalProperties
                .GetIPGlobalProperties()
                .GetActiveTcpListeners()
                .Any(endpoint => endpoint.Port == port);
        }
        catch
        {
            return false;
        }
    }
}
