===== HOSTING VHOSTS AND PHP HANDLERS =====

--- D:\Program Files\XAMPP\apache\conf\httpd.conf ---

  D:\Program Files\XAMPP\apache\conf\httpd.conf:52:# Listen: Allows you to bind Apache to specific IP addresses and/or
> D:\Program Files\XAMPP\apache\conf\httpd.conf:53:# ports, instead of the default. See also the <VirtualHost>
  D:\Program Files\XAMPP\apache\conf\httpd.conf:54:# directive.
  D:\Program Files\XAMPP\apache\conf\httpd.conf:55:#
  D:\Program Files\XAMPP\apache\conf\httpd.conf:65:# To be able to use the functionality of a module which was built as
 a DSO you
> D:\Program Files\XAMPP\apache\conf\httpd.conf:66:# have to place corresponding `LoadModule' lines at this location so
 the
  D:\Program Files\XAMPP\apache\conf\httpd.conf:67:# directives contained in it are actually available _before_ they ar
e used.
  D:\Program Files\XAMPP\apache\conf\httpd.conf:68:# Statically compiled modules (those listed by `httpd -l') do not ne
ed
  D:\Program Files\XAMPP\apache\conf\httpd.conf:71:# Example:
> D:\Program Files\XAMPP\apache\conf\httpd.conf:72:# LoadModule foo_module modules/mod_foo.so
  D:\Program Files\XAMPP\apache\conf\httpd.conf:73:#
> D:\Program Files\XAMPP\apache\conf\httpd.conf:74:LoadModule access_compat_module modules/mod_access_compat.so
> D:\Program Files\XAMPP\apache\conf\httpd.conf:75:LoadModule actions_module modules/mod_actions.so
> D:\Program Files\XAMPP\apache\conf\httpd.conf:76:LoadModule alias_module modules/mod_alias.so
> D:\Program Files\XAMPP\apache\conf\httpd.conf:77:LoadModule allowmethods_module modules/mod_allowmethods.so
> D:\Program Files\XAMPP\apache\conf\httpd.conf:78:LoadModule asis_module modules/mod_asis.so
> D:\Program Files\XAMPP\apache\conf\httpd.conf:79:LoadModule auth_basic_module modules/mod_auth_basic.so
> D:\Program Files\XAMPP\apache\conf\httpd.conf:80:#LoadModule auth_digest_module modules/mod_auth_digest.so
> D:\Program Files\XAMPP\apache\conf\httpd.conf:81:#LoadModule auth_form_module modules/mod_auth_form.so
> D:\Program Files\XAMPP\apache\conf\httpd.conf:82:#LoadModule authn_anon_module modules/mod_authn_anon.so
> D:\Program Files\XAMPP\apache\conf\httpd.conf:83:LoadModule authn_core_module modules/mod_authn_core.so
> D:\Program Files\XAMPP\apache\conf\httpd.conf:84:#LoadModule authn_dbd_module modules/mod_authn_dbd.so
> D:\Program Files\XAMPP\apache\conf\httpd.conf:85:#LoadModule authn_dbm_module modules/mod_authn_dbm.so
> D:\Program Files\XAMPP\apache\conf\httpd.conf:86:LoadModule authn_file_module modules/mod_authn_file.so
> D:\Program Files\XAMPP\apache\conf\httpd.conf:87:#LoadModule authn_socache_module modules/mod_authn_socache.so
> D:\Program Files\XAMPP\apache\conf\httpd.conf:88:#LoadModule authnz_fcgi_module modules/mod_authnz_fcgi.so
> D:\Program Files\XAMPP\apache\conf\httpd.conf:89:#LoadModule authnz_ldap_module modules/mod_authnz_ldap.so
> D:\Program Files\XAMPP\apache\conf\httpd.conf:90:LoadModule authz_core_module modules/mod_authz_core.so
> D:\Program Files\XAMPP\apache\conf\httpd.conf:91:#LoadModule authz_dbd_module modules/mod_authz_dbd.so
> D:\Program Files\XAMPP\apache\conf\httpd.conf:92:#LoadModule authz_dbm_module modules/mod_authz_dbm.so
> D:\Program Files\XAMPP\apache\conf\httpd.conf:93:LoadModule authz_groupfile_module modules/mod_authz_groupfile.so
> D:\Program Files\XAMPP\apache\conf\httpd.conf:94:LoadModule authz_host_module modules/mod_authz_host.so
> D:\Program Files\XAMPP\apache\conf\httpd.conf:95:#LoadModule authz_owner_module modules/mod_authz_owner.so
> D:\Program Files\XAMPP\apache\conf\httpd.conf:96:LoadModule authz_user_module modules/mod_authz_user.so
> D:\Program Files\XAMPP\apache\conf\httpd.conf:97:LoadModule autoindex_module modules/mod_autoindex.so
> D:\Program Files\XAMPP\apache\conf\httpd.conf:98:#LoadModule brotli_module modules/mod_brotli.so
> D:\Program Files\XAMPP\apache\conf\httpd.conf:99:#LoadModule buffer_module modules/mod_buffer.so
> D:\Program Files\XAMPP\apache\conf\httpd.conf:100:#LoadModule cache_module modules/mod_cache.so
> D:\Program Files\XAMPP\apache\conf\httpd.conf:101:#LoadModule cache_disk_module modules/mod_cache_disk.so
> D:\Program Files\XAMPP\apache\conf\httpd.conf:102:#LoadModule cache_socache_module modules/mod_cache_socache.so
> D:\Program Files\XAMPP\apache\conf\httpd.conf:103:#LoadModule cern_meta_module modules/mod_cern_meta.so
> D:\Program Files\XAMPP\apache\conf\httpd.conf:104:LoadModule cgi_module modules/mod_cgi.so
> D:\Program Files\XAMPP\apache\conf\httpd.conf:105:#LoadModule charset_lite_module modules/mod_charset_lite.so
> D:\Program Files\XAMPP\apache\conf\httpd.conf:106:#LoadModule data_module modules/mod_data.so
> D:\Program Files\XAMPP\apache\conf\httpd.conf:107:#LoadModule dav_module modules/mod_dav.so
> D:\Program Files\XAMPP\apache\conf\httpd.conf:108:#LoadModule dav_fs_module modules/mod_dav_fs.so
> D:\Program Files\XAMPP\apache\conf\httpd.conf:109:LoadModule dav_lock_module modules/mod_dav_lock.so
> D:\Program Files\XAMPP\apache\conf\httpd.conf:110:#LoadModule dbd_module modules/mod_dbd.so
> D:\Program Files\XAMPP\apache\conf\httpd.conf:111:#LoadModule deflate_module modules/mod_deflate.so
> D:\Program Files\XAMPP\apache\conf\httpd.conf:112:LoadModule dir_module modules/mod_dir.so
> D:\Program Files\XAMPP\apache\conf\httpd.conf:113:#LoadModule dumpio_module modules/mod_dumpio.so
> D:\Program Files\XAMPP\apache\conf\httpd.conf:114:LoadModule env_module modules/mod_env.so
> D:\Program Files\XAMPP\apache\conf\httpd.conf:115:#LoadModule expires_module modules/mod_expires.so
> D:\Program Files\XAMPP\apache\conf\httpd.conf:116:#LoadModule ext_filter_module modules/mod_ext_filter.so
> D:\Program Files\XAMPP\apache\conf\httpd.conf:117:#LoadModule file_cache_module modules/mod_file_cache.so
> D:\Program Files\XAMPP\apache\conf\httpd.conf:118:#LoadModule filter_module modules/mod_filter.so
> D:\Program Files\XAMPP\apache\conf\httpd.conf:119:#LoadModule http2_module modules/mod_http2.so
> D:\Program Files\XAMPP\apache\conf\httpd.conf:120:LoadModule headers_module modules/mod_headers.so
> D:\Program Files\XAMPP\apache\conf\httpd.conf:121:#LoadModule heartbeat_module modules/mod_heartbeat.so
> D:\Program Files\XAMPP\apache\conf\httpd.conf:122:#LoadModule heartmonitor_module modules/mod_heartmonitor.so
> D:\Program Files\XAMPP\apache\conf\httpd.conf:123:#LoadModule ident_module modules/mod_ident.so
> D:\Program Files\XAMPP\apache\conf\httpd.conf:124:#LoadModule imagemap_module modules/mod_imagemap.so
> D:\Program Files\XAMPP\apache\conf\httpd.conf:125:LoadModule include_module modules/mod_include.so
> D:\Program Files\XAMPP\apache\conf\httpd.conf:126:LoadModule info_module modules/mod_info.so
> D:\Program Files\XAMPP\apache\conf\httpd.conf:127:LoadModule isapi_module modules/mod_isapi.so
> D:\Program Files\XAMPP\apache\conf\httpd.conf:128:#LoadModule lbmethod_bybusyness_module modules/mod_lbmethod_bybusyn
ess.so
> D:\Program Files\XAMPP\apache\conf\httpd.conf:129:#LoadModule lbmethod_byrequests_module modules/mod_lbmethod_byreque
sts.so
> D:\Program Files\XAMPP\apache\conf\httpd.conf:130:#LoadModule lbmethod_bytraffic_module modules/mod_lbmethod_bytraffi
c.so
> D:\Program Files\XAMPP\apache\conf\httpd.conf:131:#LoadModule lbmethod_heartbeat_module modules/mod_lbmethod_heartbea
t.so
> D:\Program Files\XAMPP\apache\conf\httpd.conf:132:#LoadModule ldap_module modules/mod_ldap.so
> D:\Program Files\XAMPP\apache\conf\httpd.conf:133:#LoadModule logio_module modules/mod_logio.so
> D:\Program Files\XAMPP\apache\conf\httpd.conf:134:LoadModule log_config_module modules/mod_log_config.so
> D:\Program Files\XAMPP\apache\conf\httpd.conf:135:#LoadModule log_debug_module modules/mod_log_debug.so
> D:\Program Files\XAMPP\apache\conf\httpd.conf:136:#LoadModule log_forensic_module modules/mod_log_forensic.so
> D:\Program Files\XAMPP\apache\conf\httpd.conf:137:#LoadModule lua_module modules/mod_lua.so
> D:\Program Files\XAMPP\apache\conf\httpd.conf:138:LoadModule cache_disk_module modules/mod_cache_disk.so
> D:\Program Files\XAMPP\apache\conf\httpd.conf:139:#LoadModule macro_module modules/mod_macro.so
> D:\Program Files\XAMPP\apache\conf\httpd.conf:140:#LoadModule md_module modules/mod_md.so
> D:\Program Files\XAMPP\apache\conf\httpd.conf:141:LoadModule mime_module modules/mod_mime.so
> D:\Program Files\XAMPP\apache\conf\httpd.conf:142:#LoadModule mime_magic_module modules/mod_mime_magic.so
> D:\Program Files\XAMPP\apache\conf\httpd.conf:143:LoadModule negotiation_module modules/mod_negotiation.so
> D:\Program Files\XAMPP\apache\conf\httpd.conf:144:LoadModule proxy_module modules/mod_proxy.so
> D:\Program Files\XAMPP\apache\conf\httpd.conf:145:LoadModule proxy_ajp_module modules/mod_proxy_ajp.so
> D:\Program Files\XAMPP\apache\conf\httpd.conf:146:#LoadModule proxy_balancer_module modules/mod_proxy_balancer.so
> D:\Program Files\XAMPP\apache\conf\httpd.conf:147:#LoadModule proxy_connect_module modules/mod_proxy_connect.so
> D:\Program Files\XAMPP\apache\conf\httpd.conf:148:#LoadModule proxy_express_module modules/mod_proxy_express.so
> D:\Program Files\XAMPP\apache\conf\httpd.conf:149:LoadModule proxy_fcgi_module modules/mod_proxy_fcgi.so
> D:\Program Files\XAMPP\apache\conf\httpd.conf:150:#LoadModule proxy_ftp_module modules/mod_proxy_ftp.so
> D:\Program Files\XAMPP\apache\conf\httpd.conf:151:#LoadModule proxy_hcheck_module modules/mod_proxy_hcheck.so
> D:\Program Files\XAMPP\apache\conf\httpd.conf:152:#LoadModule proxy_html_module modules/mod_proxy_html.so
> D:\Program Files\XAMPP\apache\conf\httpd.conf:153:#LoadModule proxy_http_module modules/mod_proxy_http.so
> D:\Program Files\XAMPP\apache\conf\httpd.conf:154:#LoadModule proxy_http2_module modules/mod_proxy_http2.so
> D:\Program Files\XAMPP\apache\conf\httpd.conf:155:#LoadModule proxy_scgi_module modules/mod_proxy_scgi.so
> D:\Program Files\XAMPP\apache\conf\httpd.conf:156:#LoadModule proxy_uwsgi_module modules/mod_proxy_uwsgi.so
> D:\Program Files\XAMPP\apache\conf\httpd.conf:157:#LoadModule proxy_wstunnel_module modules/mod_proxy_wstunnel.so
> D:\Program Files\XAMPP\apache\conf\httpd.conf:158:#LoadModule ratelimit_module modules/mod_ratelimit.so
> D:\Program Files\XAMPP\apache\conf\httpd.conf:159:#LoadModule reflector_module modules/mod_reflector.so
> D:\Program Files\XAMPP\apache\conf\httpd.conf:160:#LoadModule remoteip_module modules/mod_remoteip.so
> D:\Program Files\XAMPP\apache\conf\httpd.conf:161:#LoadModule request_module modules/mod_request.so
> D:\Program Files\XAMPP\apache\conf\httpd.conf:162:#LoadModule reqtimeout_module modules/mod_reqtimeout.so
> D:\Program Files\XAMPP\apache\conf\httpd.conf:163:LoadModule rewrite_module modules/mod_rewrite.so
> D:\Program Files\XAMPP\apache\conf\httpd.conf:164:#LoadModule sed_module modules/mod_sed.so
> D:\Program Files\XAMPP\apache\conf\httpd.conf:165:#LoadModule session_module modules/mod_session.so
> D:\Program Files\XAMPP\apache\conf\httpd.conf:166:#LoadModule session_cookie_module modules/mod_session_cookie.so
> D:\Program Files\XAMPP\apache\conf\httpd.conf:167:#LoadModule session_crypto_module modules/mod_session_crypto.so
> D:\Program Files\XAMPP\apache\conf\httpd.conf:168:#LoadModule session_dbd_module modules/mod_session_dbd.so
> D:\Program Files\XAMPP\apache\conf\httpd.conf:169:LoadModule setenvif_module modules/mod_setenvif.so
> D:\Program Files\XAMPP\apache\conf\httpd.conf:170:#LoadModule slotmem_plain_module modules/mod_slotmem_plain.so
> D:\Program Files\XAMPP\apache\conf\httpd.conf:171:#LoadModule slotmem_shm_module modules/mod_slotmem_shm.so
> D:\Program Files\XAMPP\apache\conf\httpd.conf:172:#LoadModule socache_dbm_module modules/mod_socache_dbm.so
> D:\Program Files\XAMPP\apache\conf\httpd.conf:173:#LoadModule socache_memcache_module modules/mod_socache_memcache.so
> D:\Program Files\XAMPP\apache\conf\httpd.conf:174:#LoadModule socache_redis_module modules/mod_socache_redis.so
> D:\Program Files\XAMPP\apache\conf\httpd.conf:175:LoadModule socache_shmcb_module modules/mod_socache_shmcb.so
> D:\Program Files\XAMPP\apache\conf\httpd.conf:176:#LoadModule speling_module modules/mod_speling.so
> D:\Program Files\XAMPP\apache\conf\httpd.conf:177:LoadModule ssl_module modules/mod_ssl.so
> D:\Program Files\XAMPP\apache\conf\httpd.conf:178:LoadModule status_module modules/mod_status.so
> D:\Program Files\XAMPP\apache\conf\httpd.conf:179:#LoadModule substitute_module modules/mod_substitute.so
> D:\Program Files\XAMPP\apache\conf\httpd.conf:180:#LoadModule unique_id_module modules/mod_unique_id.so
> D:\Program Files\XAMPP\apache\conf\httpd.conf:181:#LoadModule userdir_module modules/mod_userdir.so
> D:\Program Files\XAMPP\apache\conf\httpd.conf:182:#LoadModule usertrack_module modules/mod_usertrack.so
> D:\Program Files\XAMPP\apache\conf\httpd.conf:183:LoadModule version_module modules/mod_version.so
> D:\Program Files\XAMPP\apache\conf\httpd.conf:184:#LoadModule vhost_alias_module modules/mod_vhost_alias.so
> D:\Program Files\XAMPP\apache\conf\httpd.conf:185:#LoadModule watchdog_module modules/mod_watchdog.so
> D:\Program Files\XAMPP\apache\conf\httpd.conf:186:#LoadModule xml2enc_module modules/mod_xml2enc.so
  D:\Program Files\XAMPP\apache\conf\httpd.conf:187:
  D:\Program Files\XAMPP\apache\conf\httpd.conf:188:<IfModule unixd_module>
  D:\Program Files\XAMPP\apache\conf\httpd.conf:205:# server, which responds to any requests that aren't handled by a
> D:\Program Files\XAMPP\apache\conf\httpd.conf:206:# <VirtualHost> definition.  These values also provide defaults for
> D:\Program Files\XAMPP\apache\conf\httpd.conf:207:# any <VirtualHost> containers you may define later in the file.
  D:\Program Files\XAMPP\apache\conf\httpd.conf:208:#
> D:\Program Files\XAMPP\apache\conf\httpd.conf:209:# All of these directives may appear inside <VirtualHost> container
s,
  D:\Program Files\XAMPP\apache\conf\httpd.conf:210:# in which case these default settings will be overridden for the
  D:\Program Files\XAMPP\apache\conf\httpd.conf:211:# virtual host being defined.
  D:\Program Files\XAMPP\apache\conf\httpd.conf:221:#
> D:\Program Files\XAMPP\apache\conf\httpd.conf:222:# ServerName gives the name and port that the server uses to identi
fy itself.
  D:\Program Files\XAMPP\apache\conf\httpd.conf:223:# This can often be determined automatically, but we recommend you
specify
  D:\Program Files\XAMPP\apache\conf\httpd.conf:224:# it explicitly to prevent problems during startup.
  D:\Program Files\XAMPP\apache\conf\httpd.conf:227:#
> D:\Program Files\XAMPP\apache\conf\httpd.conf:228:ServerName localhost:80
  D:\Program Files\XAMPP\apache\conf\httpd.conf:229:
  D:\Program Files\XAMPP\apache\conf\httpd.conf:230:#
  D:\Program Files\XAMPP\apache\conf\httpd.conf:247:#
> D:\Program Files\XAMPP\apache\conf\httpd.conf:248:# DocumentRoot: The directory out of which you will serve your
  D:\Program Files\XAMPP\apache\conf\httpd.conf:249:# documents. By default, all requests are taken from this directory
, but
  D:\Program Files\XAMPP\apache\conf\httpd.conf:250:# symbolic links and aliases may be used to point to other location
s.
  D:\Program Files\XAMPP\apache\conf\httpd.conf:251:#
> D:\Program Files\XAMPP\apache\conf\httpd.conf:252:DocumentRoot "D:/Program Files/XAMPP/htdocs"
  D:\Program Files\XAMPP\apache\conf\httpd.conf:253:<Directory "D:/Program Files/XAMPP/htdocs">
  D:\Program Files\XAMPP\apache\conf\httpd.conf:254:    #
  D:\Program Files\XAMPP\apache\conf\httpd.conf:256:    # or any combination of:
> D:\Program Files\XAMPP\apache\conf\httpd.conf:257:    #   Indexes Includes FollowSymLinks SymLinksifOwnerMatch ExecCG
I MultiViews
  D:\Program Files\XAMPP\apache\conf\httpd.conf:258:    #
  D:\Program Files\XAMPP\apache\conf\httpd.conf:259:    # Note that "MultiViews" must be named *explicitly* --- "Option
s All"
  D:\Program Files\XAMPP\apache\conf\httpd.conf:265:    #
> D:\Program Files\XAMPP\apache\conf\httpd.conf:266:    Options Indexes FollowSymLinks Includes ExecCGI
  D:\Program Files\XAMPP\apache\conf\httpd.conf:267:
  D:\Program Files\XAMPP\apache\conf\httpd.conf:268:    #
  D:\Program Files\XAMPP\apache\conf\httpd.conf:300:# ErrorLog: The location of the error log file.
> D:\Program Files\XAMPP\apache\conf\httpd.conf:301:# If you do not specify an ErrorLog directive within a <VirtualHost
>
  D:\Program Files\XAMPP\apache\conf\httpd.conf:302:# container, error messages relating to that virtual host will be
> D:\Program Files\XAMPP\apache\conf\httpd.conf:303:# logged here.  If you *do* define an error logfile for a <VirtualH
ost>
  D:\Program Files\XAMPP\apache\conf\httpd.conf:304:# container, that host's errors will be logged there and not here.
  D:\Program Files\XAMPP\apache\conf\httpd.conf:305:#
  D:\Program Files\XAMPP\apache\conf\httpd.conf:309:# LogLevel: Control the number of messages logged to the error_log.
> D:\Program Files\XAMPP\apache\conf\httpd.conf:310:# Possible values include: debug, info, notice, warn, error, crit,
  D:\Program Files\XAMPP\apache\conf\httpd.conf:311:# alert, emerg.
  D:\Program Files\XAMPP\apache\conf\httpd.conf:312:#
  D:\Program Files\XAMPP\apache\conf\httpd.conf:329:    # The location and format of the access logfile (Common Logfile
 Format).
> D:\Program Files\XAMPP\apache\conf\httpd.conf:330:    # If you do not define any access logfiles within a <VirtualHos
t>
  D:\Program Files\XAMPP\apache\conf\httpd.conf:331:    # container, they will be logged here.  Contrariwise, if you *d
o*
> D:\Program Files\XAMPP\apache\conf\httpd.conf:332:    # define per-<VirtualHost> access logfiles, transactions will b
e
  D:\Program Files\XAMPP\apache\conf\httpd.conf:333:    # logged therein and *not* in this file.
  D:\Program Files\XAMPP\apache\conf\httpd.conf:334:    #
  D:\Program Files\XAMPP\apache\conf\httpd.conf:353:    # Alias: Maps web paths into filesystem paths and is used to
> D:\Program Files\XAMPP\apache\conf\httpd.conf:354:    # access content that does not live under the DocumentRoot.
  D:\Program Files\XAMPP\apache\conf\httpd.conf:355:    # Example:
  D:\Program Files\XAMPP\apache\conf\httpd.conf:356:    # Alias /webpath /full/filesystem/path
  D:\Program Files\XAMPP\apache\conf\httpd.conf:357:    #
> D:\Program Files\XAMPP\apache\conf\httpd.conf:358:    # If you include a trailing / on /webpath then the server will
  D:\Program Files\XAMPP\apache\conf\httpd.conf:359:    # require it to be present in the URL.  You will also likely
  D:\Program Files\XAMPP\apache\conf\httpd.conf:360:    # need to provide a <Directory> section to allow access to
  D:\Program Files\XAMPP\apache\conf\httpd.conf:427:    #
> D:\Program Files\XAMPP\apache\conf\httpd.conf:428:    # AddHandler allows you to map certain file extensions to "hand
lers":
  D:\Program Files\XAMPP\apache\conf\httpd.conf:429:    # actions unrelated to filetype. These can be either built into
 the server
  D:\Program Files\XAMPP\apache\conf\httpd.conf:430:    # or added with the Action directive (see below)
  D:\Program Files\XAMPP\apache\conf\httpd.conf:434:    #
> D:\Program Files\XAMPP\apache\conf\httpd.conf:435:    AddHandler cgi-script .cgi .pl .asp
  D:\Program Files\XAMPP\apache\conf\httpd.conf:436:
  D:\Program Files\XAMPP\apache\conf\httpd.conf:437:    # For type maps (negotiated resources):
> D:\Program Files\XAMPP\apache\conf\httpd.conf:438:    #AddHandler type-map var
  D:\Program Files\XAMPP\apache\conf\httpd.conf:439:
  D:\Program Files\XAMPP\apache\conf\httpd.conf:440:    #
  D:\Program Files\XAMPP\apache\conf\httpd.conf:442:    #
> D:\Program Files\XAMPP\apache\conf\httpd.conf:443:    # To parse .shtml files for server-side includes (SSI):
> D:\Program Files\XAMPP\apache\conf\httpd.conf:444:    # (You will also need to add "Includes" to the "Options" direct
ive.)
  D:\Program Files\XAMPP\apache\conf\httpd.conf:445:    #
  D:\Program Files\XAMPP\apache\conf\httpd.conf:446:    AddType text/html .shtml
> D:\Program Files\XAMPP\apache\conf\httpd.conf:447:    AddOutputFilter INCLUDES .shtml
  D:\Program Files\XAMPP\apache\conf\httpd.conf:448:</IfModule>
  D:\Program Files\XAMPP\apache\conf\httpd.conf:449:
  D:\Program Files\XAMPP\apache\conf\httpd.conf:497:# The configuration files in the conf/extra/ directory can be
> D:\Program Files\XAMPP\apache\conf\httpd.conf:498:# included to add extra features or to modify the default configura
tion of
  D:\Program Files\XAMPP\apache\conf\httpd.conf:499:# the server, or you may simply copy their contents here and change
 as
  D:\Program Files\XAMPP\apache\conf\httpd.conf:500:# necessary.
  D:\Program Files\XAMPP\apache\conf\httpd.conf:502:# Server-pool management (MPM specific)
> D:\Program Files\XAMPP\apache\conf\httpd.conf:503:Include conf/extra/httpd-mpm.conf
  D:\Program Files\XAMPP\apache\conf\httpd.conf:504:
  D:\Program Files\XAMPP\apache\conf\httpd.conf:505:# Multi-language error messages
> D:\Program Files\XAMPP\apache\conf\httpd.conf:506:#Include conf/extra/httpd-multilang-errordoc.conf
  D:\Program Files\XAMPP\apache\conf\httpd.conf:507:
  D:\Program Files\XAMPP\apache\conf\httpd.conf:508:# Fancy directory listings
> D:\Program Files\XAMPP\apache\conf\httpd.conf:509:Include conf/extra/httpd-autoindex.conf
  D:\Program Files\XAMPP\apache\conf\httpd.conf:510:
  D:\Program Files\XAMPP\apache\conf\httpd.conf:511:# Language settings
> D:\Program Files\XAMPP\apache\conf\httpd.conf:512:Include conf/extra/httpd-languages.conf
  D:\Program Files\XAMPP\apache\conf\httpd.conf:513:
  D:\Program Files\XAMPP\apache\conf\httpd.conf:514:# User home directories
> D:\Program Files\XAMPP\apache\conf\httpd.conf:515:Include conf/extra/httpd-userdir.conf
  D:\Program Files\XAMPP\apache\conf\httpd.conf:516:
  D:\Program Files\XAMPP\apache\conf\httpd.conf:517:# Real-time info on requests and configuration
> D:\Program Files\XAMPP\apache\conf\httpd.conf:518:Include conf/extra/httpd-info.conf
  D:\Program Files\XAMPP\apache\conf\httpd.conf:519:
  D:\Program Files\XAMPP\apache\conf\httpd.conf:520:# Virtual hosts
> D:\Program Files\XAMPP\apache\conf\httpd.conf:521:Include conf/extra/httpd-vhosts.conf
  D:\Program Files\XAMPP\apache\conf\httpd.conf:522:
  D:\Program Files\XAMPP\apache\conf\httpd.conf:523:# Local access to the Apache HTTP Server Manual
> D:\Program Files\XAMPP\apache\conf\httpd.conf:524:#Include conf/extra/httpd-manual.conf
  D:\Program Files\XAMPP\apache\conf\httpd.conf:525:
  D:\Program Files\XAMPP\apache\conf\httpd.conf:526:# Distributed authoring and versioning (WebDAV)
  D:\Program Files\XAMPP\apache\conf\httpd.conf:527:#Attention! WEB_DAV is a security risk without a new userspecific c
onfiguration for a secure authentifcation
> D:\Program Files\XAMPP\apache\conf\httpd.conf:528:#Include conf/extra/httpd-dav.conf
  D:\Program Files\XAMPP\apache\conf\httpd.conf:529:
  D:\Program Files\XAMPP\apache\conf\httpd.conf:530:# Various default settings
> D:\Program Files\XAMPP\apache\conf\httpd.conf:531:#Include conf/extra/httpd-default.conf
  D:\Program Files\XAMPP\apache\conf\httpd.conf:532:# Implements a proxy/gateway for Apache.
> D:\Program Files\XAMPP\apache\conf\httpd.conf:533:Include "conf/extra/httpd-proxy.conf"
  D:\Program Files\XAMPP\apache\conf\httpd.conf:534:# Various default settings
> D:\Program Files\XAMPP\apache\conf\httpd.conf:535:Include "conf/extra/httpd-default.conf"
  D:\Program Files\XAMPP\apache\conf\httpd.conf:536:# XAMPP settings
> D:\Program Files\XAMPP\apache\conf\httpd.conf:537:Include "conf/extra/httpd-xampp.conf"
  D:\Program Files\XAMPP\apache\conf\httpd.conf:538:
  D:\Program Files\XAMPP\apache\conf\httpd.conf:539:# Configure mod_proxy_html to understand HTML4/XHTML1
  D:\Program Files\XAMPP\apache\conf\httpd.conf:540:<IfModule proxy_html_module>
> D:\Program Files\XAMPP\apache\conf\httpd.conf:541:Include conf/extra/proxy-html.conf
  D:\Program Files\XAMPP\apache\conf\httpd.conf:542:</IfModule>
  D:\Program Files\XAMPP\apache\conf\httpd.conf:543:
  D:\Program Files\XAMPP\apache\conf\httpd.conf:544:# Secure (SSL/TLS) connections
> D:\Program Files\XAMPP\apache\conf\httpd.conf:545:Include conf/extra/httpd-ssl.conf
  D:\Program Files\XAMPP\apache\conf\httpd.conf:546:#
  D:\Program Files\XAMPP\apache\conf\httpd.conf:547:# Note: The following must must be present to support
  D:\Program Files\XAMPP\apache\conf\httpd.conf:563:<IfModule mod_proxy_ajp.c>
> D:\Program Files\XAMPP\apache\conf\httpd.conf:564:Include "conf/extra/httpd-ajp.conf"
  D:\Program Files\XAMPP\apache\conf\httpd.conf:565:</IfModule>
  D:\Program Files\XAMPP\apache\conf\httpd.conf:566:</IfModule>

--- D:\Program Files\XAMPP\apache\conf\extra\httpd-vhosts.conf ---
  D:\Program Files\XAMPP\apache\conf\extra\httpd-vhosts.conf:2:
> D:\Program Files\XAMPP\apache\conf\extra\httpd-vhosts.conf:3:<VirtualHost *:80>
> D:\Program Files\XAMPP\apache\conf\extra\httpd-vhosts.conf:4:    ServerName hosting.web-engine.cc.cd
> D:\Program Files\XAMPP\apache\conf\extra\httpd-vhosts.conf:5:    DocumentRoot "D:/Program Files/XAMPP/htdocs/hosting"
  D:\Program Files\XAMPP\apache\conf\extra\httpd-vhosts.conf:6:    <Directory "D:/Program Files/XAMPP/htdocs/hosting">
  D:\Program Files\XAMPP\apache\conf\extra\httpd-vhosts.conf:7:        AllowOverride All
  D:\Program Files\XAMPP\apache\conf\extra\httpd-vhosts.conf:9:    </Directory>
> D:\Program Files\XAMPP\apache\conf\extra\httpd-vhosts.conf:10:</VirtualHost>
  D:\Program Files\XAMPP\apache\conf\extra\httpd-vhosts.conf:11:
> D:\Program Files\XAMPP\apache\conf\extra\httpd-vhosts.conf:12:<VirtualHost *:80>
> D:\Program Files\XAMPP\apache\conf\extra\httpd-vhosts.conf:13:    ServerName hosting.web-engine.info
> D:\Program Files\XAMPP\apache\conf\extra\httpd-vhosts.conf:14:    DocumentRoot "D:/Program Files/XAMPP/htdocs/hosting
"
  D:\Program Files\XAMPP\apache\conf\extra\httpd-vhosts.conf:15:    <Directory "D:/Program Files/XAMPP/htdocs/hosting">
  D:\Program Files\XAMPP\apache\conf\extra\httpd-vhosts.conf:16:        AllowOverride All
  D:\Program Files\XAMPP\apache\conf\extra\httpd-vhosts.conf:18:    </Directory>
> D:\Program Files\XAMPP\apache\conf\extra\httpd-vhosts.conf:19:</VirtualHost>
  D:\Program Files\XAMPP\apache\conf\extra\httpd-vhosts.conf:20:
> D:\Program Files\XAMPP\apache\conf\extra\httpd-vhosts.conf:21:<VirtualHost *:80>
> D:\Program Files\XAMPP\apache\conf\extra\httpd-vhosts.conf:22:    ServerName web-engine.cc.cd
> D:\Program Files\XAMPP\apache\conf\extra\httpd-vhosts.conf:23:    DocumentRoot "D:/Program Files/XAMPP/htdocs/webengi
ne"
  D:\Program Files\XAMPP\apache\conf\extra\httpd-vhosts.conf:24:    <Directory "D:/Program Files/XAMPP/htdocs/webengine
">
  D:\Program Files\XAMPP\apache\conf\extra\httpd-vhosts.conf:25:        AllowOverride All
  D:\Program Files\XAMPP\apache\conf\extra\httpd-vhosts.conf:27:    </Directory>
> D:\Program Files\XAMPP\apache\conf\extra\httpd-vhosts.conf:28:</VirtualHost>
  D:\Program Files\XAMPP\apache\conf\extra\httpd-vhosts.conf:29:
> D:\Program Files\XAMPP\apache\conf\extra\httpd-vhosts.conf:30:<VirtualHost *:80>
> D:\Program Files\XAMPP\apache\conf\extra\httpd-vhosts.conf:31:    ServerName web-engine.info
> D:\Program Files\XAMPP\apache\conf\extra\httpd-vhosts.conf:32:    DocumentRoot "D:/Program Files/XAMPP/htdocs/webengi
ne"
  D:\Program Files\XAMPP\apache\conf\extra\httpd-vhosts.conf:33:    <Directory "D:/Program Files/XAMPP/htdocs/webengine
">
  D:\Program Files\XAMPP\apache\conf\extra\httpd-vhosts.conf:34:        AllowOverride All
  D:\Program Files\XAMPP\apache\conf\extra\httpd-vhosts.conf:36:    </Directory>
> D:\Program Files\XAMPP\apache\conf\extra\httpd-vhosts.conf:37:</VirtualHost>
  D:\Program Files\XAMPP\apache\conf\extra\httpd-vhosts.conf:38:
> D:\Program Files\XAMPP\apache\conf\extra\httpd-vhosts.conf:39:<VirtualHost *:80>
> D:\Program Files\XAMPP\apache\conf\extra\httpd-vhosts.conf:40:    ServerName reshub.web-engine.info
> D:\Program Files\XAMPP\apache\conf\extra\httpd-vhosts.conf:41:    DocumentRoot "D:/Program Files/XAMPP/htdocs/reshub"
  D:\Program Files\XAMPP\apache\conf\extra\httpd-vhosts.conf:42:    <Directory "D:/Program Files/XAMPP/htdocs/reshub">
  D:\Program Files\XAMPP\apache\conf\extra\httpd-vhosts.conf:43:        AllowOverride All
  D:\Program Files\XAMPP\apache\conf\extra\httpd-vhosts.conf:45:    </Directory>
> D:\Program Files\XAMPP\apache\conf\extra\httpd-vhosts.conf:46:</VirtualHost>
  D:\Program Files\XAMPP\apache\conf\extra\httpd-vhosts.conf:47:
> D:\Program Files\XAMPP\apache\conf\extra\httpd-vhosts.conf:48:<VirtualHost *:80>
> D:\Program Files\XAMPP\apache\conf\extra\httpd-vhosts.conf:49:    ServerName reshub.web-engine.cc.cd
> D:\Program Files\XAMPP\apache\conf\extra\httpd-vhosts.conf:50:    DocumentRoot "D:/Program Files/XAMPP/htdocs/reshub"
  D:\Program Files\XAMPP\apache\conf\extra\httpd-vhosts.conf:51:    <Directory "D:/Program Files/XAMPP/htdocs/reshub">
  D:\Program Files\XAMPP\apache\conf\extra\httpd-vhosts.conf:52:        AllowOverride All
  D:\Program Files\XAMPP\apache\conf\extra\httpd-vhosts.conf:54:    </Directory>
> D:\Program Files\XAMPP\apache\conf\extra\httpd-vhosts.conf:55:</VirtualHost>
  D:\Program Files\XAMPP\apache\conf\extra\httpd-vhosts.conf:56:
> D:\Program Files\XAMPP\apache\conf\extra\httpd-vhosts.conf:57:<VirtualHost *:80>
> D:\Program Files\XAMPP\apache\conf\extra\httpd-vhosts.conf:58:    ServerName gamestrike.de5.net
> D:\Program Files\XAMPP\apache\conf\extra\httpd-vhosts.conf:59:    DocumentRoot "D:/Program Files/XAMPP/htdocs/gamestr
ike-site"
  D:\Program Files\XAMPP\apache\conf\extra\httpd-vhosts.conf:60:    <Directory "D:/Program Files/XAMPP/htdocs/gamestrik
e-site">
  D:\Program Files\XAMPP\apache\conf\extra\httpd-vhosts.conf:61:        AllowOverride All
  D:\Program Files\XAMPP\apache\conf\extra\httpd-vhosts.conf:63:    </Directory>
> D:\Program Files\XAMPP\apache\conf\extra\httpd-vhosts.conf:64:</VirtualHost>

--- D:\Program Files\XAMPP\apache\conf\extra\httpd-xampp.conf ---
  D:\Program Files\XAMPP\apache\conf\extra\httpd-xampp.conf:19:LoadFile "D:/Program Files/XAMPP/php/libsqlite3.dll"
> D:\Program Files\XAMPP\apache\conf\extra\httpd-xampp.conf:20:LoadModule php_module "D:/Program Files/XAMPP/php/php8ap
ache2_4.dll"
  D:\Program Files\XAMPP\apache\conf\extra\httpd-xampp.conf:21:
  D:\Program Files\XAMPP\apache\conf\extra\httpd-xampp.conf:22:<FilesMatch "\.php$">
> D:\Program Files\XAMPP\apache\conf\extra\httpd-xampp.conf:23:    SetHandler application/x-httpd-php
  D:\Program Files\XAMPP\apache\conf\extra\httpd-xampp.conf:24:</FilesMatch>
  D:\Program Files\XAMPP\apache\conf\extra\httpd-xampp.conf:25:<FilesMatch "\.phps$">
> D:\Program Files\XAMPP\apache\conf\extra\httpd-xampp.conf:26:    SetHandler application/x-httpd-php-source
  D:\Program Files\XAMPP\apache\conf\extra\httpd-xampp.conf:27:</FilesMatch>
  D:\Program Files\XAMPP\apache\conf\extra\httpd-xampp.conf:28:
  D:\Program Files\XAMPP\apache\conf\extra\httpd-xampp.conf:32:#<FilesMatch "\.php$">
> D:\Program Files\XAMPP\apache\conf\extra\httpd-xampp.conf:33:#    SetHandler application/x-httpd-php-cgi
  D:\Program Files\XAMPP\apache\conf\extra\httpd-xampp.conf:34:#</FilesMatch>
  D:\Program Files\XAMPP\apache\conf\extra\httpd-xampp.conf:35:#<IfModule actions_module>
  D:\Program Files\XAMPP\apache\conf\extra\httpd-xampp.conf:59:    <FilesMatch "\.php$">
> D:\Program Files\XAMPP\apache\conf\extra\httpd-xampp.conf:60:        SetHandler cgi-script
  D:\Program Files\XAMPP\apache\conf\extra\httpd-xampp.conf:61:    </FilesMatch>
  D:\Program Files\XAMPP\apache\conf\extra\httpd-xampp.conf:62:    <FilesMatch "\.phps$">
> D:\Program Files\XAMPP\apache\conf\extra\httpd-xampp.conf:63:        SetHandler None
  D:\Program Files\XAMPP\apache\conf\extra\httpd-xampp.conf:64:    </FilesMatch>
  D:\Program Files\XAMPP\apache\conf\extra\httpd-xampp.conf:65:</Directory>

===== APACHE MODULES =====
Loaded Modules:
 core_module (static)
 win32_module (static)
 mpm_winnt_module (static)
 http_module (static)
 so_module (static)
 access_compat_module (shared)
 actions_module (shared)
 alias_module (shared)
 allowmethods_module (shared)
 asis_module (shared)
 auth_basic_module (shared)
 authn_core_module (shared)
 authn_file_module (shared)
 authz_core_module (shared)
 authz_groupfile_module (shared)
 authz_host_module (shared)
 authz_user_module (shared)
 autoindex_module (shared)
 cgi_module (shared)
 dav_lock_module (shared)
 dir_module (shared)
 env_module (shared)
 headers_module (shared)
 include_module (shared)
 info_module (shared)
 isapi_module (shared)
 log_config_module (shared)
 cache_disk_module (shared)
 mime_module (shared)
 negotiation_module (shared)
 proxy_module (shared)
 proxy_ajp_module (shared)
 proxy_fcgi_module (shared)
 rewrite_module (shared)
 setenvif_module (shared)
 socache_shmcb_module (shared)
 ssl_module (shared)
 status_module (shared)
 version_module (shared)
 php_module (shared)
