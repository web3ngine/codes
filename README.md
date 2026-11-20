========================================
FILE: order.php
========================================
   1: <?php
   2: declare(strict_types=1);
   3: require_once __DIR__ . '/includes/public.php';
   4:
   5: $client = hl_require_client($pdo);
   6: $error = '';
   7: $planId = filter_input(INPUT_GET, 'plan', FILTER_VALIDATE_INT);
   8:
   9: if ($_SERVER['REQUEST_METHOD'] === 'POST') {
  10:     try {
  11:         hl_verify_csrf();
  12:
  13:         $planId = filter_var($_POST['plan_id'] ?? null, FILTER_VALIDATE_INT);
  14:         if (!$planId || $planId < 1) {
  15:             throw new RuntimeException('Invalid hosting plan.');
  16:         }
  17:
  18:         $stmt = $pdo->prepare(
  19:             "SELECT id, name, price, billing_cycle
  20:              FROM hosting_plans
  21:              WHERE id = :id AND status = 'active'
  22:              LIMIT 1"
  23:         );
  24:         $stmt->execute([':id' => $planId]);
  25:         $plan = $stmt->fetch();
  26:
  27:         if (!$plan) {
  28:             throw new RuntimeException('This plan is no longer available.');
  29:         }
  30:
  31:         $pdo->beginTransaction();
  32:
  33:         $insert = $pdo->prepare(
  34:             "INSERT INTO orders (user_id, plan_id, amount, status)
  35:              VALUES (:user_id, :plan_id, :amount, 'pending')"
  36:         );
  37:         $insert->execute([
  38:             ':user_id' => (int)$client['id'],
  39:             ':plan_id' => (int)$plan['id'],
  40:             ':amount' => $plan['price'],
  41:         ]);
  42:
  43:         $orderId = (int)$pdo->lastInsertId();
  44:         hl_log(
  45:             $pdo,
  46:             (int)$client['id'],
  47:             'client_order_created',
  48:             'Order #' . $orderId . ' created for plan ' . $plan['name'] . '. Status: pending.'
  49:         );
  50:
  51:         $pdo->commit();
  52:
  53:         header('Location: ' . hl_url('client/orders.php?created=1'));
  54:         exit;
  55:     } catch (RuntimeException $e) {
  56:         if ($pdo->inTransaction()) $pdo->rollBack();
  57:         $error = $e->getMessage();
  58:     } catch (PDOException $e) {
  59:         if ($pdo->inTransaction()) $pdo->rollBack();
  60:         error_log($e->getMessage());
  61:         $error = 'The order could not be created. Please try again.';
  62:     }
  63: }
  64:
  65: if (!$planId || $planId < 1) {
  66:     http_response_code(404);
  67:     exit('Hosting plan not found.');
  68: }
  69:
  70: $stmt = $pdo->prepare(
  71:     "SELECT * FROM hosting_plans WHERE id = :id AND status = 'active' LIMIT 1"
  72: );
  73: $stmt->execute([':id' => $planId]);
  74: $plan = $stmt->fetch();
  75:
  76: if (!$plan) {
  77:     http_response_code(404);
  78:     exit('This hosting plan is not available.');
  79: }
  80:
  81: $pageTitle = 'Confirm Order';
  82: require __DIR__ . '/includes/site_header.php';
  83: ?>
  84: <section class="page-hero compact-hero">
  85:     <div class="container">
  86:         <div class="eyebrow">ALMOST THERE</div>
  87:         <h1>Review your <span>hosting plan.</span></h1>
  88:         <p>Confirm the plan details before creating your order.</p>
  89:     </div>
  90: </section>
  91: <section class="section">
  92:     <div class="container order-layout">
  93:         <article class="price-card order-plan">
  94:             <div class="plan-kicker">SELECTED PLAN</div>
  95:             <h2><?= hl_escape($plan['name']) ?></h2>
  96:             <p class="plan-description"><?= hl_escape($plan['description'] ?: 'Hosting plan') ?></p>
  97:             <div class="price"><?= hl_money($plan['price']) ?><small>/ <?= hl_escape(hl_cycle($plan['billing_cycle'])) ?></small></div>
  98:             <ul class="plan-features">
  99:                 <?php foreach (hl_plan_specs($plan) as $spec): ?>
 100:                     <li><span>âœ“</span><?= hl_escape($spec) ?></li>
 101:                 <?php endforeach; ?>
 102:             </ul>
 103:         </article>
 104:         <div class="order-summary">
 105:             <div class="eyebrow">ORDER SUMMARY</div>
 106:             <h2>Ready to continue?</h2>
 107:             <p>Your order will be saved with a pending status. No payment is taken on this page.</p>
 108:             <div class="summary-row"><span>Account</span><strong><?= hl_escape($client['email']) ?></strong></div>
 109:             <div class="summary-row"><span>Plan</span><strong><?= hl_escape($plan['name']) ?></strong></div>
 110:             <div class="summary-row"><span>Billing</span><strong><?= hl_escape(ucfirst($plan['billing_cycle'])) ?></strong></div>
 111:             <div class="summary-total"><span>Total</span><strong><?= hl_money($plan['price']) ?></strong></div>
 112:             <?php if ($error): ?><div class="alert alert-error"><?= hl_escape($error) ?></div><?php endif; ?>
 113:             <form method="post" action="<?= hl_url('order.php?plan=' . (int)$plan['id']) ?>">
 114:                 <input type="hidden" name="csrf_token" value="<?= hl_escape(hl_csrf_token()) ?>">
 115:                 <input type="hidden" name="plan_id" value="<?= (int)$plan['id'] ?>">
 116:                 <button class="button button-full" type="submit">Create Pending Order <span>â†’</span></button>
 117:             </form>
 118:             <p class="fine-print">You can view this order in your client area.</p>
 119:         </div>
 120:     </div>
 121: </section>
 122: <?php require __DIR__ . '/includes/site_footer.php'; ?>

========================================
FILE: admin\orders.php
========================================
   1: <?php
   2: declare(strict_types=1);
   3:
   4: require_once __DIR__ . '/../includes/auth.php';
   5: require_once __DIR__ . '/../config/database.php';
   6:
   7: requireAdmin($pdo);
   8:
   9: function ordersRedirect(string $message, string $type = 'success'): never
  10: {
  11:     $_SESSION['orders_flash'] = [
  12:         'message' => $message,
  13:         'type' => $type,
  14:     ];
  15:
  16:     header('Location: orders.php');
  17:     exit;
  18: }
  19:
  20: function ordersLog(PDO $pdo, string $action, string $details): void
  21: {
  22:     $stmt = $pdo->prepare(
  23:         'INSERT INTO activity_logs (user_id, action, details, ip_address)
  24:          VALUES (:user_id, :action, :details, :ip)'
  25:     );
  26:
  27:     $stmt->execute([
  28:         ':user_id' => (int)$_SESSION['admin_id'],
  29:         ':action' => $action,
  30:         ':details' => $details,
  31:         ':ip' => $_SERVER['REMOTE_ADDR'] ?? null,
  32:     ]);
  33: }
  34:
  35: $allowedStatuses = ['pending', 'paid', 'cancelled', 'failed'];
  36:
  37: if ($_SERVER['REQUEST_METHOD'] === 'POST') {
  38:     $csrf = (string)($_POST['csrf_token'] ?? '');
  39:
  40:     if (
  41:         empty($_SESSION['admin_csrf']) ||
  42:         !hash_equals($_SESSION['admin_csrf'], $csrf)
  43:     ) {
  44:         ordersRedirect('Security token expired. Please try again.', 'error');
  45:     }
  46:
  47:     $action = (string)($_POST['action'] ?? '');
  48:
  49:     try {
  50:         if ($action !== 'update_status') {
  51:             throw new RuntimeException('Unknown action.');
  52:         }
  53:
  54:         $orderId = filter_var(
  55:             $_POST['order_id'] ?? null,
  56:             FILTER_VALIDATE_INT
  57:         );
  58:
  59:         if (!$orderId || $orderId < 1) {
  60:             throw new RuntimeException('Invalid order ID.');
  61:         }
  62:
  63:         $newStatus = (string)($_POST['status'] ?? '');
  64:
  65:         if (!in_array($newStatus, $allowedStatuses, true)) {
  66:             throw new RuntimeException('Invalid order status.');
  67:         }
  68:
  69:         $pdo->beginTransaction();
  70:
  71:         $find = $pdo->prepare(
  72:             'SELECT id, user_id, plan_id, amount, status
  73:              FROM orders
  74:              WHERE id = :id
  75:              FOR UPDATE'
  76:         );
  77:
  78:         $find->execute([':id' => $orderId]);
  79:         $order = $find->fetch();
  80:
  81:         if (!$order) {
  82:             $pdo->rollBack();
  83:             throw new RuntimeException('Order was not found.');
  84:         }
  85:
  86:         $oldStatus = (string)$order['status'];
  87:
  88:         if ($oldStatus === $newStatus) {
  89:             $pdo->rollBack();
  90:             ordersRedirect('The order already has this status.', 'error');
  91:         }
  92:
  93:         $update = $pdo->prepare(
  94:             'UPDATE orders
  95:              SET status = :status
  96:              WHERE id = :id'
  97:         );
  98:
  99:         $update->execute([
 100:             ':status' => $newStatus,
 101:             ':id' => $orderId,
 102:         ]);
 103:
 104:         ordersLog(
 105:             $pdo,
 106:             'order_status_changed',
 107:             'Order #' . $orderId .
 108:             ' status changed from ' . $oldStatus .
 109:             ' to ' . $newStatus .
 110:             '. Amount: ' . number_format((float)$order['amount'], 2, '.', '') .
 111:             '. Client ID: ' . (int)$order['user_id'] .
 112:             '. Plan ID: ' . (int)$order['plan_id'] . '.'
 113:         );
 114:
 115:         $pdo->commit();
 116:
 117:         ordersRedirect('Order #' . $orderId . ' status updated successfully.');
 118:
 119:     } catch (PDOException $e) {
 120:         if ($pdo->inTransaction()) {
 121:             $pdo->rollBack();
 122:         }
 123:
 124:         error_log($e->getMessage());
 125:         ordersRedirect('A database error occurred. Check the server error log.', 'error');
 126:
 127:     } catch (RuntimeException $e) {
 128:         if ($pdo->inTransaction()) {
 129:             $pdo->rollBack();
 130:         }
 131:
 132:         ordersRedirect($e->getMessage(), 'error');
 133:     }
 134: }
 135:
 136: /* ---------------------------------------------------------
 137:    Filters and pagination
 138: --------------------------------------------------------- */
 139:
 140: $flash = $_SESSION['orders_flash'] ?? null;
 141: unset($_SESSION['orders_flash']);
 142:
 143: $search = trim((string)($_GET['q'] ?? ''));
 144: $statusFilter = (string)($_GET['status'] ?? '');
 145:
 146: if (!in_array($statusFilter, array_merge(['all'], $allowedStatuses), true)) {
 147:     $statusFilter = 'all';
 148: }
 149:
 150: $page = max(1, (int)($_GET['page'] ?? 1));
 151: $perPage = 15;
 152:
 153: $where = 'WHERE 1 = 1';
 154: $params = [];
 155:
 156: if ($search !== '') {
 157:     $where .= "
 158:         AND (
 159:             CAST(o.id AS CHAR) LIKE :order_id
 160:             OR u.name LIKE :client_name
 161:             OR u.email LIKE :client_email
 162:             OR hp.name LIKE :plan_name
 163:         )
 164:     ";
 165:
 166:     $term = '%' . $search . '%';
 167:
 168:     $params[':order_id'] = $term;
 169:     $params[':client_name'] = $term;
 170:     $params[':client_email'] = $term;
 171:     $params[':plan_name'] = $term;
 172: }
 173:
 174: if ($statusFilter !== 'all') {
 175:     $where .= ' AND o.status = :status';
 176:     $params[':status'] = $statusFilter;
 177: }
 178:
 179: $countStmt = $pdo->prepare(
 180:     "SELECT COUNT(*)
 181:      FROM orders o
 182:      INNER JOIN users u ON u.id = o.user_id
 183:      INNER JOIN hosting_plans hp ON hp.id = o.plan_id
 184:      $where"
 185: );
 186:
 187: $countStmt->execute($params);
 188: $totalOrders = (int)$countStmt->fetchColumn();
 189:
 190: $totalPages = max(1, (int)ceil($totalOrders / $perPage));
 191: $page = min($page, $totalPages);
 192: $offset = ($page - 1) * $perPage;
 193:
 194: $listStmt = $pdo->prepare(
 195:     "SELECT
 196:         o.id,
 197:         o.user_id,
 198:         o.plan_id,
 199:         o.amount,
 200:         o.status,
 201:         o.created_at,
 202:         u.name AS client_name,
 203:         u.email AS client_email,
 204:         hp.name AS plan_name,
 205:         hp.billing_cycle
 206:      FROM orders o
 207:      INNER JOIN users u ON u.id = o.user_id
 208:      INNER JOIN hosting_plans hp ON hp.id = o.plan_id
 209:      $where
 210:      ORDER BY o.id DESC
 211:      LIMIT :limit OFFSET :offset"
 212: );
 213:
 214: foreach ($params as $key => $value) {
 215:     $listStmt->bindValue($key, $value, PDO::PARAM_STR);
 216: }
 217:
 218: $listStmt->bindValue(':limit', $perPage, PDO::PARAM_INT);
 219: $listStmt->bindValue(':offset', $offset, PDO::PARAM_INT);
 220: $listStmt->execute();
 221:
 222: $orders = $listStmt->fetchAll();
 223:
 224: /* ---------------------------------------------------------
 225:    Dashboard statistics
 226: --------------------------------------------------------- */
 227:
 228: $stats = $pdo->query(
 229:     "SELECT
 230:         COUNT(*) AS total,
 231:         SUM(status = 'pending') AS pending_count,
 232:         SUM(status = 'paid') AS paid_count,
 233:         SUM(status = 'cancelled') AS cancelled_count,
 234:         SUM(status = 'failed') AS failed_count,
 235:         COALESCE(SUM(CASE WHEN status = 'paid' THEN amount ELSE 0 END), 0) AS revenue
 236:      FROM orders"
 237: )->fetch();
 238:
 239: /* ---------------------------------------------------------
 240:    Selected order details
 241: --------------------------------------------------------- */
 242:
 243: $selectedOrder = null;
 244: $viewId = filter_var($_GET['view'] ?? null, FILTER_VALIDATE_INT);
 245:
 246: if ($viewId && $viewId > 0) {
 247:     $detailStmt = $pdo->prepare(
 248:         "SELECT
 249:             o.id,
 250:             o.user_id,
 251:             o.plan_id,
 252:             o.amount,
 253:             o.status,
 254:             o.created_at,
 255:             u.name AS client_name,
 256:             u.email AS client_email,
 257:             u.status AS client_status,
 258:             hp.name AS plan_name,
 259:             hp.description AS plan_description,
 260:             hp.price AS plan_price,
 261:             hp.billing_cycle,
 262:             hp.disk_limit_mb,
 263:             hp.bandwidth_limit_mb,
 264:             hp.domains_limit,
 265:             hp.databases_limit,
 266:             hp.ftp_accounts_limit
 267:          FROM orders o
 268:          INNER JOIN users u ON u.id = o.user_id
 269:          INNER JOIN hosting_plans hp ON hp.id = o.plan_id
 270:          WHERE o.id = :id
 271:          LIMIT 1"
 272:     );
 273:
 274:     $detailStmt->execute([':id' => $viewId]);
 275:     $selectedOrder = $detailStmt->fetch() ?: null;
 276: }
 277:
 278: function ordersPageUrl(int $page, string $search, string $status): string
 279: {
 280:     $query = ['page' => $page];
 281:
 282:     if ($search !== '') {
 283:         $query['q'] = $search;
 284:     }
 285:
 286:     if ($status !== 'all') {
 287:         $query['status'] = $status;
 288:     }
 289:
 290:     return 'orders.php?' . http_build_query($query);
 291: }
 292:
 293: function orderStatusLabel(string $status): string
 294: {
 295:     return match ($status) {
 296:         'pending' => 'Pending',
 297:         'paid' => 'Paid',
 298:         'cancelled' => 'Cancelled',
 299:         'failed' => 'Failed',
 300:         default => ucfirst($status),
 301:     };
 302: }
 303:
 304: ?>
 305: <!DOCTYPE html>
 306: <html lang="en">
 307: <head>
 308: <meta charset="UTF-8">
 309: <meta name="viewport" content="width=device-width, initial-scale=1.0">
 310: <title>Orders | Hosting Lab</title>
 311:
 312: <style>
 313: :root {
 314:     color-scheme: dark;
 315:     --bg: #090d16;
 316:     --panel: #111827;
 317:     --panel2: #172235;
 318:     --border: #263449;
 319:     --text: #edf2fa;
 320:     --muted: #91a0b5;
 321:     --accent: #6c8cff;
 322:     --green: #35d49a;
 323:     --red: #ff7185;
 324:     --yellow: #f5c76b;
 325: }
 326:
 327: * { box-sizing: border-box; }
 328:
 329: body {
 330:     margin: 0;
 331:     min-height: 100vh;
 332:     background:
 333:         radial-gradient(circle at 85% 0%, #17264a 0, transparent 32%),
 334:         var(--bg);
 335:     color: var(--text);
 336:     font-family: Inter, "Segoe UI", Arial, sans-serif;
 337: }
 338:
 339: .layout { display: flex; min-height: 100vh; }
 340:
 341: .sidebar {
 342:     width: 245px;
 343:     padding: 28px 18px;
 344:     background: rgba(12,18,30,.96);
 345:     border-right: 1px solid var(--border);
 346:     flex-shrink: 0;
 347: }
 348:
 349: .brand {
 350:     font-size: 21px;
 351:     font-weight: 800;
 352:     margin: 0 10px 8px;
 353: }
 354:
 355: .brand span { color: var(--accent); }
 356:
 357: .brand-subtitle {
 358:     color: var(--muted);
 359:     font-size: 11px;
 360:     margin: 0 10px 38px;
 361:     letter-spacing: 1.5px;
 362:     text-transform: uppercase;
 363: }
 364:
 365: .nav-label {
 366:     color: #63738a;
 367:     font-size: 10px;
 368:     font-weight: 700;
 369:     letter-spacing: 1.5px;
 370:     margin: 25px 10px 12px;
 371:     text-transform: uppercase;
 372: }
 373:
 374: .nav-link {
 375:     display: block;
 376:     padding: 12px 13px;
 377:     margin: 5px 0;
 378:     color: var(--muted);
 379:     text-decoration: none;
 380:     border-radius: 9px;
 381:     font-size: 13px;
 382: }
 383:
 384: .nav-link:hover, .nav-link.active {
 385:     color: white;
 386:     background: #202e49;
 387: }
 388:
 389: .nav-link.active { box-shadow: inset 3px 0 var(--accent); }
 390:
 391: .main {
 392:     flex: 1;
 393:     min-width: 0;
 394:     padding: 35px;
 395: }
 396:
 397: .topbar {
 398:     display: flex;
 399:     justify-content: space-between;
 400:     align-items: center;
 401:     gap: 20px;
 402:     margin-bottom: 30px;
 403: }
 404:
 405: .breadcrumb {
 406:     color: var(--muted);
 407:     font-size: 12px;
 408:     margin-bottom: 9px;
 409: }
 410:
 411: h1 {
 412:     margin: 0;
 413:     font-size: 29px;
 414:     letter-spacing: -1px;
 415: }
 416:
 417: .subtitle {
 418:     color: var(--muted);
 419:     margin-top: 9px;
 420:     font-size: 13px;
 421: }
 422:
 423: .environment {
 424:     padding: 9px 13px;
 425:     border: 1px solid #235b4a;
 426:     background: #102c26;
 427:     color: var(--green);
 428:     border-radius: 30px;
 429:     font-size: 11px;
 430:     white-space: nowrap;
 431: }
 432:
 433: .environment::before {
 434:     content: "";
 435:     display: inline-block;
 436:     width: 7px;
 437:     height: 7px;
 438:     margin-right: 8px;
 439:     background: var(--green);
 440:     border-radius: 50%;
 441: }
 442:
 443: .stats {
 444:     display: grid;
 445:     grid-template-columns: repeat(4, minmax(0, 1fr));
 446:     gap: 15px;
 447:     margin-bottom: 25px;
 448: }
 449:
 450: .stat-card {
 451:     padding: 20px;
 452:     border: 1px solid var(--border);
 453:     border-radius: 13px;
 454:     background: linear-gradient(145deg, #152137, #101725);
 455: }
 456:
 457: .stat-label { color: var(--muted); font-size: 12px; }
 458:
 459: .stat-value {
 460:     margin-top: 12px;
 461:     font-size: 26px;
 462:     font-weight: 750;
 463: }
 464:
 465: .stat-value.green { color: var(--green); }
 466: .stat-value.yellow { color: var(--yellow); }
 467: .stat-value.red { color: var(--red); }
 468:
 469: .panel {
 470:     border: 1px solid var(--border);
 471:     border-radius: 14px;
 472:     overflow: hidden;
 473:     background: rgba(17,24,39,.94);
 474:     margin-bottom: 24px;
 475: }
 476:
 477: .panel-header {
 478:     padding: 20px 23px;
 479:     border-bottom: 1px solid var(--border);
 480: }
 481:
 482: .panel-header h2 {
 483:     margin: 0;
 484:     font-size: 16px;
 485: }
 486:
 487: .panel-header p {
 488:     margin: 6px 0 0;
 489:     color: var(--muted);
 490:     font-size: 12px;
 491: }
 492:
 493: .flash {
 494:     padding: 14px 17px;
 495:     margin-bottom: 20px;
 496:     border: 1px solid var(--border);
 497:     border-radius: 9px;
 498:     font-size: 13px;
 499: }
 500:
 501: .flash.success {
 502:     background: #102c26;
 503:     border-color: #235b4a;
 504:     color: var(--green);
 505: }
 506:
 507: .flash.error {
 508:     background: #351b25;
 509:     border-color: #69303f;
 510:     color: #ff9aaa;
 511: }
 512:
 513: .notice {
 514:     padding: 13px 16px;
 515:     margin: 18px 22px;
 516:     border: 1px solid #66502a;
 517:     border-radius: 9px;
 518:     background: #2b2417;
 519:     color: #f5d18a;
 520:     font-size: 12px;
 521:     line-height: 1.6;
 522: }
 523:
 524: .toolbar {
 525:     display: flex;
 526:     justify-content: space-between;
 527:     align-items: center;
 528:     gap: 14px;
 529:     padding: 18px 22px;
 530:     flex-wrap: wrap;
 531: }
 532:
 533: .filters {
 534:     display: flex;
 535:     gap: 8px;
 536:     flex: 1;
 537:     flex-wrap: wrap;
 538: }
 539:
 540: .filters input,
 541: .filters select {
 542:     padding: 10px 12px;
 543:     border: 1px solid var(--border);
 544:     border-radius: 8px;
 545:     background: #0b1321;
 546:     color: var(--text);
 547:     outline: none;
 548:     font: inherit;
 549:     font-size: 12px;
 550: }
 551:
 552: .filters input {
 553:     flex: 1;
 554:     min-width: 180px;
 555: }
 556:
 557: .filters input:focus,
 558: .filters select:focus {
 559:     border-color: var(--accent);
 560: }
 561:
 562: button, .button {
 563:     display: inline-flex;
 564:     align-items: center;
 565:     justify-content: center;
 566:     border: 0;
 567:     border-radius: 8px;
 568:     padding: 10px 14px;
 569:     font-size: 12px;
 570:     font-weight: 700;
 571:     cursor: pointer;
 572:     text-decoration: none;
 573:     transition: .15s;
 574: }
 575:
 576: .primary { background: var(--accent); color: white; }
 577: .primary:hover { background: #829dff; }
 578:
 579: .secondary { background: #263449; color: var(--text); }
 580: .secondary:hover { background: #344660; }
 581:
 582: .danger { background: #49232e; color: #ff9aaa; }
 583: .success { background: #123c31; color: #71e7b7; }
 584: .warning { background: #49391d; color: #f5d18a; }
 585:
 586: .table-wrap { overflow-x: auto; }
 587:
 588: table {
 589:     width: 100%;
 590:     border-collapse: collapse;
 591:     min-width: 1050px;
 592: }
 593:
 594: th {
 595:     padding: 15px 17px;
 596:     text-align: left;
 597:     color: #8191a8;
 598:     background: #111b2b;
 599:     font-size: 10px;
 600:     letter-spacing: 1px;
 601:     text-transform: uppercase;
 602: }
 603:
 604: td {
 605:     padding: 16px 17px;
 606:     border-top: 1px solid #202c3e;
 607:     font-size: 12px;
 608:     white-space: nowrap;
 609: }
 610:
 611: tr:hover td { background: rgba(108,140,255,.035); }
 612:
 613: .order-id { font-weight: 750; color: #dce5ff; }
 614:
 615: .client-name { font-weight: 700; font-size: 12px; }
 616:
 617: .client-email {
 618:     color: var(--muted);
 619:     font-size: 11px;
 620:     margin-top: 5px;
 621: }
 622:
 623: .plan-name { font-weight: 650; }
 624:
 625: .amount { font-weight: 750; color: #dce5ff; }
 626:
 627: .status {
 628:     display: inline-block;
 629:     padding: 6px 10px;
 630:     border-radius: 20px;
 631:     font-size: 10px;
 632:     font-weight: 700;
 633: }
 634:
 635: .status.pending { color: var(--yellow); background: #49391d; }
 636: .status.paid { color: var(--green); background: #123329; }
 637: .status.cancelled { color: #c1c9d6; background: #293345; }
 638: .status.failed { color: var(--red); background: #3b2029; }
 639:
 640: .actions {
 641:     display: flex;
 642:     align-items: center;
 643:     gap: 6px;
 644: }
 645:
 646: .actions form { display: inline; }
 647: .actions button { padding: 8px 10px; font-size: 10px; }
 648:
 649: .status-form {
 650:     display: flex;
 651:     align-items: center;
 652:     gap: 6px;
 653: }
 654:
 655: .status-form select {
 656:     max-width: 112px;
 657:     padding: 8px;
 658:     border: 1px solid var(--border);
 659:     border-radius: 7px;
 660:     background: #0b1321;
 661:     color: var(--text);
 662:     font-size: 10px;
 663: }
 664:
 665: .status-form button {
 666:     padding: 8px 10px;
 667:     font-size: 10px;
 668: }
 669:
 670: .detail-grid {
 671:     display: grid;
 672:     grid-template-columns: repeat(3, minmax(0, 1fr));
 673:     gap: 15px;
 674:     padding: 22px;
 675: }
 676:
 677: .detail-item {
 678:     padding: 15px;
 679:     border: 1px solid var(--border);
 680:     border-radius: 9px;
 681:     background: #0d1625;
 682:     min-width: 0;
 683: }
 684:
 685: .detail-label {
 686:     color: var(--muted);
 687:     font-size: 10px;
 688:     text-transform: uppercase;
 689:     letter-spacing: .8px;
 690: }
 691:
 692: .detail-value {
 693:     margin-top: 8px;
 694:     font-size: 13px;
 695:     font-weight: 650;
 696:     overflow-wrap: anywhere;
 697: }
 698:
 699: .pagination {
 700:     display: flex;
 701:     align-items: center;
 702:     justify-content: space-between;
 703:     gap: 12px;
 704:     padding: 17px 22px;
 705:     border-top: 1px solid var(--border);
 706:     color: var(--muted);
 707:     font-size: 11px;
 708:     flex-wrap: wrap;
 709: }
 710:
 711: .pagination-links {
 712:     display: flex;
 713:     gap: 7px;
 714: }
 715:
 716: .pagination-links a {
 717:     padding: 8px 11px;
 718:     border: 1px solid var(--border);
 719:     border-radius: 7px;
 720:     color: var(--text);
 721:     text-decoration: none;
 722: }
 723:
 724: .pagination-links a.current {
 725:     background: #26385d;
 726:     border-color: var(--accent);
 727: }
 728:
 729: .empty {
 730:     padding: 35px;
 731:     text-align: center;
 732:     color: var(--muted);
 733: }
 734:
 735: .footer {
 736:     color: #66758a;
 737:     font-size: 11px;
 738:     margin-top: 22px;
 739:     text-align: right;
 740: }
 741:
 742: @media (max-width: 1100px) {
 743:     .stats { grid-template-columns: repeat(2, minmax(0, 1fr)); }
 744:     .detail-grid { grid-template-columns: repeat(2, minmax(0, 1fr)); }
 745: }
 746:
 747: @media (max-width: 900px) {
 748:     .sidebar { width: 190px; }
 749:     .main { padding: 25px 18px; }
 750: }
 751:
 752: @media (max-width: 650px) {
 753:     .sidebar { display: none; }
 754:     .stats, .detail-grid { grid-template-columns: 1fr; }
 755:     .topbar { align-items: flex-start; flex-direction: column; }
 756:     .toolbar { align-items: stretch; }
 757: }
 758: </style>
 759: </head>
 760:
 761: <body>
 762: <div class="layout">
 763:
 764: <aside class="sidebar">
 765:     <div class="brand">Hosting<span>Lab</span></div>
 766:     <div class="brand-subtitle">Administration</div>
 767:
 768:     <div class="nav-label">Overview</div>
 769:     <a class="nav-link" href="index.php">Dashboard</a>
 770:
 771:     <div class="nav-label">Management</div>
 772:     <a class="nav-link" href="plans.php">Hosting Plans</a>
 773:     <a class="nav-link" href="services.php">Services</a>
 774:     <a class="nav-link active" href="orders.php">Orders</a>
 775:     <a class="nav-link" href="clients.php">Clients</a>
 776:
 777:     <div class="nav-label">System</div>
 778:     <a class="nav-link" href="logs.php">Activity Logs</a>
 779:     <a class="nav-link" href="settings.php">Settings</a>
 780: </aside>
 781:
 782: <main class="main">
 783:
 784:     <div class="topbar">
 785:         <div>
 786:             <div class="breadcrumb">Administration / Orders</div>
 787:             <h1>Orders Management</h1>
 788:             <div class="subtitle">Review customer orders and manage their current status.</div>
 789:         </div>
 790:
 791:         <div class="environment">LOCAL ENVIRONMENT</div>
 792:     </div>
 793:
 794:     <?php if (is_array($flash)): ?>
 795:         <div class="flash <?= ($flash['type'] ?? '') === 'error' ? 'error' : 'success' ?>">
 796:             <?= adminEscape($flash['message'] ?? '') ?>
 797:         </div>
 798:     <?php endif; ?>
 799:
 800:     <section class="stats">
 801:         <div class="stat-card">
 802:             <div class="stat-label">Total Orders</div>
 803:             <div class="stat-value"><?= number_format((int)($stats['total'] ?? 0)) ?></div>
 804:         </div>
 805:
 806:         <div class="stat-card">
 807:             <div class="stat-label">Pending Orders</div>
 808:             <div class="stat-value yellow"><?= number_format((int)($stats['pending_count'] ?? 0)) ?></div>
 809:         </div>
 810:
 811:         <div class="stat-card">
 812:             <div class="stat-label">Paid Orders</div>
 813:             <div class="stat-value green"><?= number_format((int)($stats['paid_count'] ?? 0)) ?></div>
 814:         </div>
 815:
 816:         <div class="stat-card">
 817:             <div class="stat-label">Recorded Revenue</div>
 818:             <div class="stat-value green">â‚¬<?= number_format((float)($stats['revenue'] ?? 0), 2) ?></div>
 819:         </div>
 820:     </section>
 821:
 822:     <section class="panel">
 823:         <div class="panel-header">
 824:             <h2>Order Records</h2>
 825:             <p>Orders are loaded directly from the Hosting Lab database.</p>
 826:         </div>
 827:
 828:         <div class="notice">
 829:             Changing an order to Paid here is a manual administrative status update.
 830:             It does not verify a payment or create a hosting service.
 831:         </div>
 832:
 833:         <div class="toolbar">
 834:             <form class="filters" method="get" action="orders.php">
 835:                 <input
 836:                     type="search"
 837:                     name="q"
 838:                     value="<?= adminEscape($search) ?>"
 839:                     placeholder="Search order ID, client, email or plan..."
 840:                 >
 841:
 842:                 <select name="status">
 843:                     <option value="all" <?= $statusFilter === 'all' ? 'selected' : '' ?>>All statuses</option>
 844:                     <option value="pending" <?= $statusFilter === 'pending' ? 'selected' : '' ?>>Pending</option>
 845:                     <option value="paid" <?= $statusFilter === 'paid' ? 'selected' : '' ?>>Paid</option>
 846:                     <option value="cancelled" <?= $statusFilter === 'cancelled' ? 'selected' : '' ?>>Cancelled</option>
 847:                     <option value="failed" <?= $statusFilter === 'failed' ? 'selected' : '' ?>>Failed</option>
 848:                 </select>
 849:
 850:                 <button type="submit" class="primary">Apply Filters</button>
 851:
 852:                 <?php if ($search !== '' || $statusFilter !== 'all'): ?>
 853:                     <a class="button secondary" href="orders.php">Clear</a>
 854:                 <?php endif; ?>
 855:             </form>
 856:
 857:             <div class="stat-label">
 858:                 <?= number_format($totalOrders) ?> matching order(s)
 859:             </div>
 860:         </div>
 861:
 862:         <?php if (!$orders): ?>
 863:             <div class="empty">
 864:                 <?= $search !== '' || $statusFilter !== 'all'
 865:                     ? 'No orders match the selected filters.'
 866:                     : 'No orders have been created yet.' ?>
 867:             </div>
 868:         <?php else: ?>
 869:             <div class="table-wrap">
 870:                 <table>
 871:                     <thead>
 872:                         <tr>
 873:                             <th>Order</th>
 874:                             <th>Client</th>
 875:                             <th>Hosting Plan</th>
 876:                             <th>Amount</th>
 877:                             <th>Status</th>
 878:                             <th>Created</th>
 879:                             <th>Actions</th>
 880:                         </tr>
 881:                     </thead>
 882:
 883:                     <tbody>
 884:                     <?php foreach ($orders as $order): ?>
 885:                         <tr>
 886:                             <td>
 887:                                 <span class="order-id">#<?= (int)$order['id'] ?></span>
 888:                             </td>
 889:
 890:                             <td>
 891:                                 <div class="client-name"><?= adminEscape($order['client_name']) ?></div>
 892:                                 <div class="client-email"><?= adminEscape($order['client_email']) ?></div>
 893:                             </td>
 894:
 895:                             <td>
 896:                                 <div class="plan-name"><?= adminEscape($order['plan_name']) ?></div>
 897:                                 <div class="client-email"><?= adminEscape(ucfirst($order['billing_cycle'])) ?></div>
 898:                             </td>
 899:
 900:                             <td>
 901:                                 <span class="amount">â‚¬<?= number_format((float)$order['amount'], 2) ?></span>
 902:                             </td>
 903:
 904:                             <td>
 905:                                 <span class="status <?= adminEscape($order['status']) ?>">
 906:                                     <?= adminEscape(orderStatusLabel($order['status'])) ?>
 907:                                 </span>
 908:                             </td>
 909:
 910:                             <td>
 911:                                 <?= adminEscape(date('d M Y H:i', strtotime($order['created_at']))) ?>
 912:                             </td>
 913:
 914:                             <td>
 915:                                 <div class="actions">
 916:                                     <a
 917:                                         class="button secondary"
 918:                                         href="orders.php?view=<?= (int)$order['id'] ?>"
 919:                                     >View</a>
 920:
 921:                                     <form
 922:                                         class="status-form"
 923:                                         method="post"
 924:                                         action="orders.php"
 925:                                         onsubmit="return confirm('Change the status of order #<?= (int)$order['id'] ?>?')"
 926:                                     >
 927:                                         <input type="hidden" name="csrf_token" value="<?= adminEscape($_SESSION['admin_csrf']) ?>">
 928:                                         <input type="hidden" name="action" value="update_status">
 929:                                         <input type="hidden" name="order_id" value="<?= (int)$order['id'] ?>">
 930:
 931:                                         <select name="status" aria-label="New order status">
 932:                                             <?php foreach ($allowedStatuses as $status): ?>
 933:                                                 <option
 934:                                                     value="<?= adminEscape($status) ?>"
 935:                                                     <?= $order['status'] === $status ? 'selected' : '' ?>
 936:                                                 >
 937:                                                     <?= adminEscape(orderStatusLabel($status)) ?>
 938:                                                 </option>
 939:                                             <?php endforeach; ?>
 940:                                         </select>
 941:
 942:                                         <button type="submit" class="primary">Save</button>
 943:                                     </form>
 944:                                 </div>
 945:                             </td>
 946:                         </tr>
 947:                     <?php endforeach; ?>
 948:                     </tbody>
 949:                 </table>
 950:             </div>
 951:
 952:             <div class="pagination">
 953:                 <div>Page <?= $page ?> of <?= $totalPages ?></div>
 954:
 955:                 <div class="pagination-links">
 956:                     <?php if ($page > 1): ?>
 957:                         <a href="<?= adminEscape(ordersPageUrl($page - 1, $search, $statusFilter)) ?>">Previous</a>
 958:                     <?php endif; ?>
 959:
 960:                     <?php
 961:                     $startPage = max(1, $page - 2);
 962:                     $endPage = min($totalPages, $page + 2);
 963:                     ?>
 964:
 965:                     <?php for ($pageNumber = $startPage; $pageNumber <= $endPage; $pageNumber++): ?>
 966:                         <a
 967:                             class="<?= $pageNumber === $page ? 'current' : '' ?>"
 968:                             href="<?= adminEscape(ordersPageUrl($pageNumber, $search, $statusFilter)) ?>"
 969:                         ><?= $pageNumber ?></a>
 970:                     <?php endfor; ?>
 971:
 972:                     <?php if ($page < $totalPages): ?>
 973:                         <a href="<?= adminEscape(ordersPageUrl($page + 1, $search, $statusFilter)) ?>">Next</a>
 974:                     <?php endif; ?>
 975:                 </div>
 976:             </div>
 977:         <?php endif; ?>
 978:     </section>
 979:
 980:     <?php if ($viewId): ?>
 981:         <section class="panel">
 982:             <div class="panel-header">
 983:                 <h2>Order Details</h2>
 984:                 <p>Full information for the selected order.</p>
 985:             </div>
 986:
 987:             <?php if (!$selectedOrder): ?>
 988:                 <div class="empty">The selected order could not be found.</div>
 989:             <?php else: ?>
 990:                 <div class="detail-grid">
 991:                     <div class="detail-item">
 992:                         <div class="detail-label">Order ID</div>
 993:                         <div class="detail-value">#<?= (int)$selectedOrder['id'] ?></div>
 994:                     </div>
 995:
 996:                     <div class="detail-item">
 997:                         <div class="detail-label">Order Amount</div>
 998:                         <div class="detail-value">â‚¬<?= number_format((float)$selectedOrder['amount'], 2) ?></div>
 999:                     </div>
1000:
1001:                     <div class="detail-item">
1002:                         <div class="detail-label">Order Status</div>
1003:                         <div class="detail-value">
1004:                             <span class="status <?= adminEscape($selectedOrder['status']) ?>">
1005:                                 <?= adminEscape(orderStatusLabel($selectedOrder['status'])) ?>
1006:                             </span>
1007:                         </div>
1008:                     </div>
1009:
1010:                     <div class="detail-item">
1011:                         <div class="detail-label">Client Name</div>
1012:                         <div class="detail-value"><?= adminEscape($selectedOrder['client_name']) ?></div>
1013:                     </div>
1014:
1015:                     <div class="detail-item">
1016:                         <div class="detail-label">Client Email</div>
1017:                         <div class="detail-value"><?= adminEscape($selectedOrder['client_email']) ?></div>
1018:                     </div>
1019:
1020:                     <div class="detail-item">
1021:                         <div class="detail-label">Client Account Status</div>
1022:                         <div class="detail-value"><?= adminEscape(ucfirst($selectedOrder['client_status'])) ?></div>
1023:                     </div>
1024:
1025:                     <div class="detail-item">
1026:                         <div class="detail-label">Hosting Plan</div>
1027:                         <div class="detail-value"><?= adminEscape($selectedOrder['plan_name']) ?></div>
1028:                     </div>
1029:
1030:                     <div class="detail-item">
1031:                         <div class="detail-label">Plan Billing Cycle</div>
1032:                         <div class="detail-value"><?= adminEscape(ucfirst($selectedOrder['billing_cycle'])) ?></div>
1033:                     </div>
1034:
1035:                     <div class="detail-item">
1036:                         <div class="detail-label">Plan Listed Price</div>
1037:                         <div class="detail-value">â‚¬<?= number_format((float)$selectedOrder['plan_price'], 2) ?></div>
1038:                     </div>
1039:
1040:                     <div class="detail-item">
1041:                         <div class="detail-label">Client ID</div>
1042:                         <div class="detail-value">#<?= (int)$selectedOrder['user_id'] ?></div>
1043:                     </div>
1044:
1045:                     <div class="detail-item">
1046:                         <div class="detail-label">Plan ID</div>
1047:                         <div class="detail-value">#<?= (int)$selectedOrder['plan_id'] ?></div>
1048:                     </div>
1049:
1050:                     <div class="detail-item">
1051:                         <div class="detail-label">Created At</div>
1052:                         <div class="detail-value"><?= adminEscape(date('d M Y H:i:s', strtotime($selectedOrder['created_at']))) ?></div>
1053:                     </div>
1054:
1055:                     <div class="detail-item">
1056:                         <div class="detail-label">SSD Storage</div>
1057:                         <div class="detail-value"><?= number_format((int)$selectedOrder['disk_limit_mb'] / 1024, 2) ?> GB</div>
1058:                     </div>
1059:
1060:                     <div class="detail-item">
1061:                         <div class="detail-label">Bandwidth</div>
1062:                         <div class="detail-value">
1063:                             <?= (int)$selectedOrder['bandwidth_limit_mb'] === 0
1064:                                 ? 'Unlimited'
1065:                                 : number_format((int)$selectedOrder['bandwidth_limit_mb'] / 1024, 2) . ' GB' ?>
1066:                         </div>
1067:                     </div>
1068:
1069:                     <div class="detail-item">
1070:                         <div class="detail-label">Domains</div>
1071:                         <div class="detail-value"><?= number_format((int)$selectedOrder['domains_limit']) ?></div>
1072:                     </div>
1073:
1074:                     <div class="detail-item">
1075:                         <div class="detail-label">Databases</div>
1076:                         <div class="detail-value">
1077:                             <?= (int)$selectedOrder['databases_limit'] >= 400
1078:                                 ? 'Unlimited'
1079:                                 : number_format((int)$selectedOrder['databases_limit']) ?>
1080:                         </div>
1081:                     </div>
1082:
1083:                     <div class="detail-item">
1084:                         <div class="detail-label">FTP Accounts</div>
1085:                         <div class="detail-value"><?= number_format((int)$selectedOrder['ftp_accounts_limit']) ?></div>
1086:                     </div>
1087:                 </div>
1088:
1089:                 <?php if (!empty($selectedOrder['plan_description'])): ?>
1090:                     <div class="notice">
1091:                         <?= nl2br(adminEscape($selectedOrder['plan_description'])) ?>
1092:                     </div>
1093:                 <?php endif; ?>
1094:
1095:                 <div style="padding: 0 22px 22px;">
1096:                     <a class="button secondary" href="orders.php">Close Details</a>
1097:                 </div>
1098:             <?php endif; ?>
1099:         </section>
1100:     <?php endif; ?>
1101:
1102:     <div class="footer">
1103:         Hosting Lab Â· Orders Management
1104:     </div>
1105:
1106: </main>
1107: </div>
1108: </body>
1109: </html>

========================================
FILE: admin\services.php
========================================
   1: <?php
   2: declare(strict_types=1);
   3:
   4: require_once __DIR__ . '/../includes/auth.php';
   5: require_once __DIR__ . '/../config/database.php';
   6:
   7: requireAdmin($pdo);
   8:
   9: function servicesRedirect(string $message, string $type = 'success'): never
  10: {
  11:     $_SESSION['services_flash'] = [
  12:         'message' => $message,
  13:         'type' => $type
  14:     ];
  15:
  16:     header('Location: services.php');
  17:     exit;
  18: }
  19:
  20: function servicesLog(PDO $pdo, string $action, string $details): void
  21: {
  22:     $stmt = $pdo->prepare(
  23:         'INSERT INTO activity_logs (user_id, action, details, ip_address)
  24:          VALUES (:user_id, :action, :details, :ip)'
  25:     );
  26:
  27:     $stmt->execute([
  28:         ':user_id' => (int)$_SESSION['admin_id'],
  29:         ':action' => $action,
  30:         ':details' => $details,
  31:         ':ip' => $_SERVER['REMOTE_ADDR'] ?? null
  32:     ]);
  33: }
  34:
  35: function servicesPostString(string $key): string
  36: {
  37:     return trim((string)($_POST[$key] ?? ''));
  38: }
  39:
  40: function servicesEscape(mixed $value): string
  41: {
  42:     return htmlspecialchars((string)$value, ENT_QUOTES, 'UTF-8');
  43: }
  44:
  45: $allowedStatuses = ['pending', 'active', 'suspended', 'terminated'];
  46:
  47: if ($_SERVER['REQUEST_METHOD'] === 'POST') {
  48:     $csrf = (string)($_POST['csrf_token'] ?? '');
  49:
  50:     if (
  51:         empty($_SESSION['admin_csrf']) ||
  52:         !hash_equals($_SESSION['admin_csrf'], $csrf)
  53:     ) {
  54:         servicesRedirect('Security token expired. Please try again.', 'error');
  55:     }
  56:
  57:     $action = servicesPostString('action');
  58:     $id = filter_var($_POST['id'] ?? null, FILTER_VALIDATE_INT);
  59:
  60:     if (!$id || $id < 1) {
  61:         servicesRedirect('Invalid service ID.', 'error');
  62:     }
  63:
  64:     try {
  65:         $stmt = $pdo->prepare(
  66:             'SELECT id, domain, username, status
  67:              FROM hosting_services
  68:              WHERE id = :id
  69:              LIMIT 1'
  70:         );
  71:         $stmt->execute([':id' => $id]);
  72:         $service = $stmt->fetch();
  73:
  74:         if (!$service) {
  75:             throw new RuntimeException('Service not found.');
  76:         }
  77:
  78:         if ($action === 'update') {
  79:             $domain = servicesPostString('domain');
  80:             $username = servicesPostString('username');
  81:
  82:             if (
  83:                 $domain === '' ||
  84:                 mb_strlen($domain) > 190 ||
  85:                 preg_match('/[\s\/\\\\]/', $domain)
  86:             ) {
  87:                 throw new RuntimeException('Enter a valid domain name.');
  88:             }
  89:
  90:             if (
  91:                 $username === '' ||
  92:                 mb_strlen($username) > 100
  93:             ) {
  94:                 throw new RuntimeException('Username is required and must not exceed 100 characters.');
  95:             }
  96:
  97:             $pdo->beginTransaction();
  98:
  99:             $update = $pdo->prepare(
 100:                 'UPDATE hosting_services
 101:                  SET domain = :domain, username = :username
 102:                  WHERE id = :id'
 103:             );
 104:
 105:             $update->execute([
 106:                 ':domain' => $domain,
 107:                 ':username' => $username,
 108:                 ':id' => $id
 109:             ]);
 110:
 111:             servicesLog(
 112:                 $pdo,
 113:                 'hosting_service_updated',
 114:                 'Service #' . $id . ' updated. Domain: ' .
 115:                 $service['domain'] . ' -> ' . $domain
 116:             );
 117:
 118:             $pdo->commit();
 119:
 120:             servicesRedirect('Service details updated successfully.');
 121:         }
 122:
 123:         if ($action === 'status') {
 124:             $status = servicesPostString('status');
 125:
 126:             if (!in_array($status, $allowedStatuses, true)) {
 127:                 throw new RuntimeException('Invalid service status.');
 128:             }
 129:
 130:             if ($service['status'] === 'terminated') {
 131:                 throw new RuntimeException('Terminated services cannot be reactivated.');
 132:             }
 133:
 134:             $pdo->beginTransaction();
 135:
 136:             $update = $pdo->prepare(
 137:                 'UPDATE hosting_services
 138:                  SET status = :status
 139:                  WHERE id = :id'
 140:             );
 141:
 142:             $update->execute([
 143:                 ':status' => $status,
 144:                 ':id' => $id
 145:             ]);
 146:
 147:             servicesLog(
 148:                 $pdo,
 149:                 'hosting_service_status_changed',
 150:                 'Service #' . $id . ' status changed from ' .
 151:                 $service['status'] . ' to ' . $status
 152:             );
 153:
 154:             $pdo->commit();
 155:
 156:             servicesRedirect('Service status updated successfully.');
 157:         }
 158:
 159:         throw new RuntimeException('Unknown action.');
 160:
 161:     } catch (Throwable $e) {
 162:         if ($pdo->inTransaction()) {
 163:             $pdo->rollBack();
 164:         }
 165:
 166:         error_log('Hosting services error: ' . $e->getMessage());
 167:
 168:         servicesRedirect(
 169:             $e instanceof RuntimeException
 170:                 ? $e->getMessage()
 171:                 : 'An unexpected error occurred. Check the application log.',
 172:             'error'
 173:         );
 174:     }
 175: }
 176:
 177: $flash = $_SESSION['services_flash'] ?? null;
 178: unset($_SESSION['services_flash']);
 179:
 180: $search = trim((string)($_GET['search'] ?? ''));
 181: $statusFilter = (string)($_GET['status'] ?? '');
 182:
 183: if (!in_array($statusFilter, array_merge([''], $allowedStatuses), true)) {
 184:     $statusFilter = '';
 185: }
 186:
 187: $page = max(1, (int)($_GET['page'] ?? 1));
 188: $perPage = 15;
 189: $offset = ($page - 1) * $perPage;
 190:
 191: $where = [];
 192: $params = [];
 193:
 194: if ($search !== '') {
 195:     $where[] = '(hs.domain LIKE :search_domain
 196:                  OR hs.username LIKE :search_username
 197:                  OR u.name LIKE :search_name
 198:                  OR u.email LIKE :search_email
 199:                  OR CAST(hs.id AS CHAR) = :search_id)';
 200:
 201:     $term = '%' . $search . '%';
 202:
 203:     $params[':search_domain'] = $term;
 204:     $params[':search_username'] = $term;
 205:     $params[':search_name'] = $term;
 206:     $params[':search_email'] = $term;
 207:     $params[':search_id'] = $search;
 208: }
 209:
 210: if ($statusFilter !== '') {
 211:     $where[] = 'hs.status = :status';
 212:     $params[':status'] = $statusFilter;
 213: }
 214:
 215: $whereSql = $where ? 'WHERE ' . implode(' AND ', $where) : '';
 216:
 217: $countStmt = $pdo->prepare(
 218:     "SELECT COUNT(*)
 219:      FROM hosting_services hs
 220:      INNER JOIN users u ON u.id = hs.user_id
 221:      INNER JOIN hosting_plans hp ON hp.id = hs.plan_id
 222:      $whereSql"
 223: );
 224:
 225: $countStmt->execute($params);
 226: $totalServices = (int)$countStmt->fetchColumn();
 227: $totalPages = max(1, (int)ceil($totalServices / $perPage));
 228:
 229: $page = min($page, $totalPages);
 230: $offset = ($page - 1) * $perPage;
 231:
 232: $listStmt = $pdo->prepare(
 233:     "SELECT
 234:         hs.id,
 235:         hs.user_id,
 236:         hs.plan_id,
 237:         hs.order_id,
 238:         hs.domain,
 239:         hs.username,
 240:         hs.status,
 241:         hs.created_at,
 242:         u.name AS client_name,
 243:         u.email AS client_email,
 244:         hp.name AS plan_name,
 245:         o.status AS order_status
 246:      FROM hosting_services hs
 247:      INNER JOIN users u ON u.id = hs.user_id
 248:      INNER JOIN hosting_plans hp ON hp.id = hs.plan_id
 249:      LEFT JOIN orders o ON o.id = hs.order_id
 250:      $whereSql
 251:      ORDER BY hs.id DESC
 252:      LIMIT $perPage OFFSET $offset"
 253: );
 254:
 255: $listStmt->execute($params);
 256: $services = $listStmt->fetchAll();
 257:
 258: $stats = $pdo->query(
 259:     "SELECT
 260:         COUNT(*) AS total,
 261:         SUM(status = 'active') AS active,
 262:         SUM(status = 'pending') AS pending,
 263:         SUM(status = 'suspended') AS suspended,
 264:         SUM(status = 'terminated') AS terminated_count
 265:      FROM hosting_services"
 266: )->fetch();
 267:
 268: $baseQuery = [];
 269:
 270: if ($search !== '') {
 271:     $baseQuery['search'] = $search;
 272: }
 273:
 274: if ($statusFilter !== '') {
 275:     $baseQuery['status'] = $statusFilter;
 276: }
 277:
 278: $queryString = http_build_query($baseQuery);
 279:
 280: ?>
 281: <!DOCTYPE html>
 282: <html lang="en">
 283: <head>
 284: <meta charset="UTF-8">
 285: <meta name="viewport" content="width=device-width, initial-scale=1.0">
 286: <title>Hosting Services | Hosting Lab</title>
 287: <style>
 288: :root {
 289:     --bg:#0b1020;
 290:     --panel:#121a2d;
 291:     --panel2:#182238;
 292:     --border:#26334d;
 293:     --text:#edf2ff;
 294:     --muted:#8e9bb5;
 295:     --accent:#6685ff;
 296:     --green:#36c995;
 297:     --yellow:#e9b95d;
 298:     --red:#ef737d;
 299: }
 300: *{box-sizing:border-box}
 301: body{
 302:     margin:0;background:var(--bg);color:var(--text);
 303:     font-family:Inter,Segoe UI,Arial,sans-serif;
 304: }
 305: .layout{display:flex;min-height:100vh}
 306: .sidebar{
 307:     width:245px;background:#0e1628;border-right:1px solid var(--border);
 308:     padding:28px 18px;flex-shrink:0;
 309: }
 310: .brand{font-size:21px;font-weight:800;margin-bottom:8px}
 311: .brand span{color:var(--accent)}
 312: .brand-sub{font-size:11px;color:var(--muted);margin-bottom:35px}
 313: .nav-label{
 314:     color:#63718d;font-size:10px;font-weight:700;
 315:     letter-spacing:1.5px;margin:24px 10px 10px;
 316: }
 317: .nav-link{
 318:     display:block;text-decoration:none;color:#aab6cc;
 319:     padding:12px 14px;border-radius:9px;margin:4px 0;font-size:13px;
 320: }
 321: .nav-link:hover,.nav-link.active{
 322:     background:#202d49;color:#fff;
 323: }
 324: .nav-link.active{box-shadow:inset 3px 0 var(--accent)}
 325: .main{flex:1;min-width:0;padding:32px}
 326: .topbar{
 327:     display:flex;justify-content:space-between;align-items:center;
 328:     gap:15px;margin-bottom:30px;
 329: }
 330: h1{font-size:27px;margin:0 0 7px}
 331: .subtitle{font-size:13px;color:var(--muted)}
 332: .admin{font-size:12px;color:var(--muted)}
 333: .stats{
 334:     display:grid;grid-template-columns:repeat(5,minmax(0,1fr));
 335:     gap:14px;margin-bottom:24px;
 336: }
 337: .stat,.panel{
 338:     background:var(--panel);border:1px solid var(--border);
 339:     border-radius:13px;
 340: }
 341: .stat{padding:19px}
 342: .stat-label{font-size:11px;color:var(--muted);margin-bottom:12px}
 343: .stat-value{font-size:25px;font-weight:750}
 344: .panel{padding:20px;margin-bottom:20px}
 345: .panel-title{font-size:15px;font-weight:700;margin-bottom:17px}
 346: .filters{
 347:     display:flex;gap:10px;flex-wrap:wrap;align-items:center
 348: }
 349: input,select{
 350:     background:#0d1526;color:var(--text);
 351:     border:1px solid var(--border);border-radius:8px;
 352:     padding:11px 12px;font-size:12px;outline:none;
 353: }
 354: input{min-width:220px;flex:1}
 355: select{min-width:145px}
 356: button,.button{
 357:     border:0;border-radius:8px;padding:11px 15px;
 358:     background:var(--accent);color:white;font-size:12px;
 359:     font-weight:700;cursor:pointer;text-decoration:none;
 360: }
 361: button:hover,.button:hover{filter:brightness(1.12)}
 362: .secondary{background:#263550}
 363: .danger{background:#74343e}
 364: .table-wrap{overflow-x:auto}
 365: table{width:100%;border-collapse:collapse;min-width:950px}
 366: th{
 367:     color:#8391aa;text-align:left;font-size:10px;
 368:     text-transform:uppercase;letter-spacing:.8px;
 369:     padding:13px 10px;border-bottom:1px solid var(--border);
 370: }
 371: td{
 372:     padding:15px 10px;border-bottom:1px solid #202c42;
 373:     font-size:12px;vertical-align:middle;
 374: }
 375: tr:last-child td{border-bottom:0}
 376: .domain{font-weight:700;color:#e8edff}
 377: .small{font-size:11px;color:var(--muted);margin-top:5px}
 378: .badge{
 379:     display:inline-block;padding:6px 9px;border-radius:20px;
 380:     font-size:10px;font-weight:700;text-transform:capitalize;
 381: }
 382: .active{background:#123b34;color:#57dfb2}
 383: .pending{background:#45391e;color:#f0cb77}
 384: .suspended{background:#462b35;color:#ff9ba6}
 385: .terminated{background:#34394a;color:#b0bad0}
 386: .actions{display:flex;gap:7px;flex-wrap:wrap}
 387: .actions button{padding:8px 10px}
 388: .flash{
 389:     padding:13px 15px;border-radius:9px;margin-bottom:18px;
 390:     font-size:12px;background:#15372f;color:#70e1b9;
 391: }
 392: .flash.error{background:#46242d;color:#ffabb2}
 393: .pagination{
 394:     display:flex;justify-content:space-between;align-items:center;
 395:     margin-top:18px;color:var(--muted);font-size:12px;
 396: }
 397: .pagination a{color:#aebfff;text-decoration:none;margin-left:12px}
 398: .empty{text-align:center;padding:45px;color:var(--muted)}
 399: .footer{color:#64728c;font-size:11px;margin-top:25px}
 400: @media(max-width:1050px){
 401:     .stats{grid-template-columns:repeat(3,1fr)}
 402:     .sidebar{width:205px}
 403: }
 404: @media(max-width:700px){
 405:     .layout{display:block}
 406:     .sidebar{width:100%;padding:17px}
 407:     .brand-sub{margin-bottom:12px}
 408:     .nav-label{display:none}
 409:     .nav-link{display:inline-block;padding:9px}
 410:     .main{padding:20px 13px}
 411:     .stats{grid-template-columns:repeat(2,1fr)}
 412:     .topbar{align-items:flex-start;flex-direction:column}
 413: }
 414: </style>
 415: </head>
 416: <body>
 417: <div class="layout">
 418: <aside class="sidebar">
 419:     <div class="brand">HOSTING <span>LAB</span></div>
 420:     <div class="brand-sub">ADMIN CONTROL PANEL</div>
 421:
 422:     <div class="nav-label">MAIN MENU</div>
 423:     <a class="nav-link" href="index.php">Dashboard</a>
 424:     <a class="nav-link" href="plans.php">Hosting Plans</a>
 425:     <a class="nav-link active" href="services.php">Services</a>
 426:     <a class="nav-link" href="orders.php">Orders</a>
 427:     <a class="nav-link" href="clients.php">Clients</a>
 428:
 429:     <div class="nav-label">SYSTEM</div>
 430:     <a class="nav-link" href="logs.php">Activity Logs</a>
 431:     <a class="nav-link" href="settings.php">Settings</a>
 432:     <a class="nav-link" href="logout.php">Logout</a>
 433: </aside>
 434:
 435: <main class="main">
 436:     <header class="topbar">
 437:         <div>
 438:             <h1>Hosting Services</h1>
 439:             <div class="subtitle">Manage client hosting services and service status.</div>
 440:         </div>
 441:         <div class="admin">
 442:             Administrator: <?= servicesEscape($_SESSION['admin_name'] ?? 'Admin') ?>
 443:         </div>
 444:     </header>
 445:
 446:     <?php if ($flash): ?>
 447:         <div class="flash <?= ($flash['type'] ?? '') === 'error' ? 'error' : '' ?>">
 448:             <?= servicesEscape($flash['message'] ?? '') ?>
 449:         </div>
 450:     <?php endif; ?>
 451:
 452:     <section class="stats">
 453:         <div class="stat">
 454:             <div class="stat-label">TOTAL SERVICES</div>
 455:             <div class="stat-value"><?= (int)$stats['total'] ?></div>
 456:         </div>
 457:         <div class="stat">
 458:             <div class="stat-label">ACTIVE</div>
 459:             <div class="stat-value"><?= (int)$stats['active'] ?></div>
 460:         </div>
 461:         <div class="stat">
 462:             <div class="stat-label">PENDING</div>
 463:             <div class="stat-value"><?= (int)$stats['pending'] ?></div>
 464:         </div>
 465:         <div class="stat">
 466:             <div class="stat-label">SUSPENDED</div>
 467:             <div class="stat-value"><?= (int)$stats['suspended'] ?></div>
 468:         </div>
 469:         <div class="stat">
 470:             <div class="stat-label">TERMINATED</div>
 471:             <div class="stat-value"><?= (int)$stats['terminated_count'] ?></div>
 472:         </div>
 473:     </section>
 474:
 475:     <section class="panel">
 476:         <div class="panel-title">Search and filters</div>
 477:         <form class="filters" method="get">
 478:             <input
 479:                 type="search"
 480:                 name="search"
 481:                 placeholder="Search domain, username, client or ID..."
 482:                 value="<?= servicesEscape($search) ?>"
 483:             >
 484:             <select name="status">
 485:                 <option value="">All statuses</option>
 486:                 <?php foreach ($allowedStatuses as $status): ?>
 487:                     <option value="<?= servicesEscape($status) ?>"
 488:                         <?= $statusFilter === $status ? 'selected' : '' ?>>
 489:                         <?= ucfirst($status) ?>
 490:                     </option>
 491:                 <?php endforeach; ?>
 492:             </select>
 493:             <button type="submit">Apply filters</button>
 494:             <a class="button secondary" href="services.php">Reset</a>
 495:         </form>
 496:     </section>
 497:
 498:     <section class="panel">
 499:         <div class="panel-title">Services directory</div>
 500:
 501:         <div class="table-wrap">
 502:         <table>
 503:             <thead>
 504:                 <tr>
 505:                     <th>Service</th>
 506:                     <th>Client</th>
 507:                     <th>Hosting Plan</th>
 508:                     <th>Order</th>
 509:                     <th>Status</th>
 510:                     <th>Created</th>
 511:                     <th>Actions</th>
 512:                 </tr>
 513:             </thead>
 514:             <tbody>
 515:             <?php if (!$services): ?>
 516:                 <tr>
 517:                     <td colspan="7" class="empty">No hosting services found.</td>
 518:                 </tr>
 519:             <?php else: ?>
 520:                 <?php foreach ($services as $service): ?>
 521:                 <tr>
 522:                     <td>
 523:                         <div class="domain"><?= servicesEscape($service['domain']) ?></div>
 524:                         <div class="small">
 525:                             #<?= (int)$service['id'] ?> Â·
 526:                             <?= servicesEscape($service['username']) ?>
 527:                         </div>
 528:                     </td>
 529:                     <td>
 530:                         <strong><?= servicesEscape($service['client_name']) ?></strong>
 531:                         <div class="small"><?= servicesEscape($service['client_email']) ?></div>
 532:                     </td>
 533:                     <td><?= servicesEscape($service['plan_name']) ?></td>
 534:                     <td>
 535:                         <?= $service['order_id'] !== null
 536:                             ? '#' . (int)$service['order_id']
 537:                             : 'â€”' ?>
 538:                         <?php if ($service['order_status']): ?>
 539:                             <div class="small"><?= servicesEscape($service['order_status']) ?></div>
 540:                         <?php endif; ?>
 541:                     </td>
 542:                     <td>
 543:                         <span class="badge <?= servicesEscape($service['status']) ?>">
 544:                             <?= servicesEscape($service['status']) ?>
 545:                         </span>
 546:                     </td>
 547:                     <td><?= servicesEscape($service['created_at']) ?></td>
 548:                     <td>
 549:                         <div class="actions">
 550:                             <button type="button" class="secondary"
 551:                                 onclick="editService(
 552:                                     <?= (int)$service['id'] ?>,
 553:                                     <?= htmlspecialchars(json_encode($service['domain']), ENT_QUOTES, 'UTF-8') ?>,
 554:                                     <?= htmlspecialchars(json_encode($service['username']), ENT_QUOTES, 'UTF-8') ?>
 555:                                 )">Edit</button>
 556:
 557:                             <?php if ($service['status'] !== 'terminated'): ?>
 558:                             <form method="post" onsubmit="return confirm('Change status of this service?')">
 559:                                 <input type="hidden" name="csrf_token"
 560:                                     value="<?= servicesEscape($_SESSION['admin_csrf']) ?>">
 561:                                 <input type="hidden" name="action" value="status">
 562:                                 <input type="hidden" name="id" value="<?= (int)$service['id'] ?>">
 563:                                 <input type="hidden" name="status"
 564:                                     value="<?= $service['status'] === 'active' ? 'suspended' : 'active' ?>">
 565:                                 <button type="submit" class="<?= $service['status'] === 'active' ? 'danger' : '' ?>">
 566:                                     <?= $service['status'] === 'active' ? 'Suspend' : 'Activate' ?>
 567:                                 </button>
 568:                             </form>
 569:
 570:                             <form method="post" onsubmit="return confirm('Terminate this service permanently? This cannot be undone here.')">
 571:                                 <input type="hidden" name="csrf_token"
 572:                                     value="<?= servicesEscape($_SESSION['admin_csrf']) ?>">
 573:                                 <input type="hidden" name="action" value="status">
 574:                                 <input type="hidden" name="id" value="<?= (int)$service['id'] ?>">
 575:                                 <input type="hidden" name="status" value="terminated">
 576:                                 <button type="submit" class="danger">Terminate</button>
 577:                             </form>
 578:                             <?php endif; ?>
 579:                         </div>
 580:                     </td>
 581:                 </tr>
 582:                 <?php endforeach; ?>
 583:             <?php endif; ?>
 584:             </tbody>
 585:         </table>
 586:         </div>
 587:
 588:         <div class="pagination">
 589:             <span>
 590:                 Showing <?= count($services) ?> of <?= $totalServices ?> services
 591:             </span>
 592:             <span>
 593:                 <?php if ($page > 1): ?>
 594:                     <a href="?<?= servicesEscape($queryString) ?>&page=<?= $page - 1 ?>">Previous</a>
 595:                 <?php endif; ?>
 596:                 Page <?= $page ?> / <?= $totalPages ?>
 597:                 <?php if ($page < $totalPages): ?>
 598:                     <a href="?<?= servicesEscape($queryString) ?>&page=<?= $page + 1 ?>">Next</a>
 599:                 <?php endif; ?>
 600:             </span>
 601:         </div>
 602:     </section>
 603:
 604:     <div class="footer">Hosting Lab Â· Services Management</div>
 605: </main>
 606: </div>
 607:
 608: <div id="editModal" style="display:none;position:fixed;inset:0;background:#000b;z-index:10;align-items:center;justify-content:center;padding:20px">
 609:     <div class="panel" style="width:100%;max-width:460px">
 610:         <div class="panel-title">Edit hosting service</div>
 611:         <form method="post">
 612:             <input type="hidden" name="csrf_token"
 613:                 value="<?= servicesEscape($_SESSION['admin_csrf']) ?>">
 614:             <input type="hidden" name="action" value="update">
 615:             <input type="hidden" name="id" id="editId">
 616:
 617:             <label class="small">Domain</label>
 618:             <input style="width:100%;margin:8px 0 16px" name="domain" id="editDomain" maxlength="190" required>
 619:
 620:             <label class="small">Username</label>
 621:             <input style="width:100%;margin:8px 0 18px" name="username" id="editUsername" maxlength="100" required>
 622:
 623:             <div class="actions">
 624:                 <button type="submit">Save changes</button>
 625:                 <button type="button" class="secondary" onclick="closeEdit()">Cancel</button>
 626:             </div>
 627:         </form>
 628:     </div>
 629: </div>
 630:
 631: <script>
 632: function editService(id, domain, username) {
 633:     document.getElementById('editId').value = id;
 634:     document.getElementById('editDomain').value = domain;
 635:     document.getElementById('editUsername').value = username;
 636:     document.getElementById('editModal').style.display = 'flex';
 637: }
 638:
 639: function closeEdit() {
 640:     document.getElementById('editModal').style.display = 'none';
 641: }
 642:
 643: document.getElementById('editModal').addEventListener('click', function(event) {
 644:     if (event.target === this) closeEdit();
 645: });
 646: </script>
 647: </body>
 648: </html>

========================================
FILE: includes\auth.php
========================================
   1: <?php
   2: declare(strict_types=1);
   3:
   4: if (session_status() !== PHP_SESSION_ACTIVE) {
   5:     $isHttps = (
   6:         isset($_SERVER['HTTPS']) &&
   7:         $_SERVER['HTTPS'] !== '' &&
   8:         $_SERVER['HTTPS'] !== 'off'
   9:     );
  10:
  11:     session_set_cookie_params([
  12:         'lifetime' => 0,
  13:         'path' => '/hosting-lab/',
  14:         'secure' => $isHttps,
  15:         'httponly' => true,
  16:         'samesite' => 'Lax',
  17:     ]);
  18:
  19:     session_start();
  20: }
  21:
  22: function adminIsAuthenticated(PDO $pdo): bool
  23: {
  24:     $adminId = filter_var(
  25:         $_SESSION['admin_id'] ?? null,
  26:         FILTER_VALIDATE_INT
  27:     );
  28:
  29:     if (!$adminId) {
  30:         return false;
  31:     }
  32:
  33:     $stmt = $pdo->prepare(
  34:         "SELECT id, name, email
  35:          FROM users
  36:          WHERE id = :id
  37:            AND role = 'admin'
  38:            AND status = 'active'
  39:          LIMIT 1"
  40:     );
  41:
  42:     $stmt->execute([':id' => $adminId]);
  43:     $admin = $stmt->fetch();
  44:
  45:     if (!$admin) {
  46:         unset(
  47:             $_SESSION['admin_id'],
  48:             $_SESSION['admin_name'],
  49:             $_SESSION['admin_email'],
  50:             $_SESSION['admin_csrf']
  51:         );
  52:
  53:         return false;
  54:     }
  55:
  56:     $_SESSION['admin_name'] = $admin['name'];
  57:     $_SESSION['admin_email'] = $admin['email'];
  58:
  59:     return true;
  60: }
  61:
  62: function requireAdmin(PDO $pdo): void
  63: {
  64:     if (!adminIsAuthenticated($pdo)) {
  65:         header('Location: login.php');
  66:         exit;
  67:     }
  68:
  69:     if (empty($_SESSION['admin_csrf'])) {
  70:         $_SESSION['admin_csrf'] = bin2hex(random_bytes(32));
  71:     }
  72: }
  73:
  74: function adminEscape(mixed $value): string
  75: {
  76:     return htmlspecialchars(
  77:         (string)$value,
  78:         ENT_QUOTES,
  79:         'UTF-8'
  80:     );
  81: }

========================================
FILE: client\services.php
========================================
   1: <?php
   2: declare(strict_types=1);
   3: require_once __DIR__ . '/../includes/public.php';
   4:
   5: $client = hl_require_client($pdo);
   6: $pageTitle = 'My Services';
   7:
   8: $stmt = $pdo->prepare(
   9:     'SELECT hs.id, hs.domain, hs.username, hs.status, hs.created_at, hp.name AS plan_name
  10:      FROM hosting_services hs INNER JOIN hosting_plans hp ON hp.id = hs.plan_id
  11:      WHERE hs.user_id = :id ORDER BY hs.id DESC'
  12: );
  13: $stmt->execute([':id' => $client['id']]);
  14: $services = $stmt->fetchAll();
  15:
  16: require __DIR__ . '/../includes/site_header.php';
  17: ?>
  18: <section class="dashboard-hero"><div class="container dashboard-heading"><div><div class="eyebrow">CLIENT AREA</div><h1>My <span>services.</span></h1><p>View the hosting services assigned to your account.</p></div><a class="button" href="<?= hl_url('plans.php') ?>">Browse Plans <span>â†’</span></a></div></section>
  19: <section class="section dashboard-section"><div class="container">
  20:     <div class="dashboard-nav"><a href="<?= hl_url('client/index.php') ?>">Overview</a><a href="<?= hl_url('client/orders.php') ?>">My Orders</a><a class="dash-active" href="<?= hl_url('client/services.php') ?>">My Services</a><a href="<?= hl_url('cpanel/index.php') ?>">Hosting Control Panel</a><a href="<?= hl_url('logout.php') ?>">Logout</a></div>
  21:     <div class="content-card"><div class="card-heading"><div><div class="eyebrow">HOSTING SERVICES</div><h2>Assigned services</h2></div></div>
  22:     <?php if ($services): ?>
  23:     <div class="table-wrap"><table class="data-table"><thead><tr><th>Service</th><th>Plan</th><th>Domain</th><th>Username</th><th>Status</th><th>Created</th></tr></thead><tbody>
  24:     <?php foreach ($services as $service): ?><tr><td>#<?= (int)$service['id'] ?></td><td><?= hl_escape($service['plan_name']) ?></td><td><?= hl_escape($service['domain']) ?></td><td><?= hl_escape($service['username']) ?></td><td><span class="status status-<?= hl_escape($service['status']) ?>"><?= hl_escape(ucfirst($service['status'])) ?></span></td><td><?= hl_escape($service['created_at']) ?></td></tr><?php endforeach; ?>
  25:     </tbody></table></div>
  26:     <?php else: ?><div class="empty-state">No hosting services have been assigned to your account yet. New services will appear here when provisioned.</div><?php endif; ?>
  27:     </div>
  28: </div></section>
  29: <?php require __DIR__ . '/../includes/site_footer.php'; ?>

========================================
FILE: cpanel\_module.php
========================================
   1: <?php
   2: declare(strict_types=1);
   3:
   4: require_once __DIR__ . '/includes/bootstrap.php';
   5:
   6: $pageTitle = $moduleTitle ?? 'Hosting Tool';
   7: $activePage = $moduleKey ?? '';
   8: $moduleDescription = $moduleDescription ?? '';
   9:
  10: require __DIR__ . '/includes/header.php';
  11: ?>
  12:
  13: <section class="page-intro">
  14:     <span class="eyebrow">WEB-ENGINE HOSTING TOOLS</span>
  15:     <h2><?= hl_escape($pageTitle) ?></h2>
  16:     <p><?= hl_escape($moduleDescription) ?></p>
  17: </section>
  18:
  19: <section class="content-panel module-placeholder">
  20:     <div class="module-icon">âŒ˜</div>
  21:     <span class="status status-pending">NOT CONFIGURED</span>
  22:     <h3>This module is not connected yet</h3>
  23:     <p>
  24:         The interface is ready. The secure hosting backend for this feature
  25:         must be implemented before this tool can make changes to your account.
  26:     </p>
  27:     <a class="secondary-button" href="<?= hl_url('cpanel/index.php') ?>">â† Back to Dashboard</a>
  28: </section>
  29:
  30: <?php require __DIR__ . '/includes/footer.php'; ?>

========================================
FILE: cpanel\services.php
========================================
   1: <?php
   2: declare(strict_types=1);
   3:
   4: require_once __DIR__ . '/includes/bootstrap.php';
   5:
   6: $pageTitle = 'My Services';
   7: $activePage = 'services';
   8:
   9: $stmt = $pdo->prepare(
  10:     'SELECT hs.id, hs.domain, hs.username, hs.status,
  11:             hp.name AS plan_name,
  12:             hp.disk_limit_mb, hp.bandwidth_limit_mb,
  13:             hp.domains_limit, hp.databases_limit,
  14:             hp.ftp_accounts_limit
  15:      FROM hosting_services hs
  16:      INNER JOIN hosting_plans hp ON hp.id = hs.plan_id
  17:      WHERE hs.user_id = :id
  18:      ORDER BY hs.id DESC'
  19: );
  20: $stmt->execute([':id' => $client['id']]);
  21: $services = $stmt->fetchAll();
  22:
  23: require __DIR__ . '/includes/header.php';
  24: ?>
  25:
  26: <section class="page-intro">
  27:     <span class="eyebrow">ACCOUNT RESOURCES</span>
  28:     <h2>My Hosting Services</h2>
  29:     <p>View the hosting services assigned to your account.</p>
  30: </section>
  31:
  32: <section class="content-panel">
  33:     <div class="panel-heading">
  34:         <div>
  35:             <span class="eyebrow">SERVICE LIST</span>
  36:             <h2><?= count($services) ?> service(s)</h2>
  37:         </div>
  38:     </div>
  39:
  40:     <?php if ($services): ?>
  41:         <div class="service-grid">
  42:         <?php foreach ($services as $service): ?>
  43:             <article class="service-card">
  44:                 <div class="service-card-top">
  45:                     <span class="service-symbol">â—ˆ</span>
  46:                     <span class="status status-<?= hl_escape($service['status']) ?>">
  47:                         <?= hl_escape(ucfirst($service['status'])) ?>
  48:                     </span>
  49:                 </div>
  50:
  51:                 <span class="eyebrow">SERVICE #<?= (int)$service['id'] ?></span>
  52:                 <h3><?= hl_escape($service['domain']) ?></h3>
  53:                 <p class="service-plan"><?= hl_escape($service['plan_name']) ?> Plan</p>
  54:
  55:                 <div class="service-details">
  56:                     <div><span>Username</span><strong><?= hl_escape($service['username']) ?></strong></div>
  57:                     <div><span>SSD Storage</span><strong><?= number_format((float)$service['disk_limit_mb'] / 1024, 1) ?> GB</strong></div>
  58:                     <div><span>Bandwidth</span><strong><?= (int)$service['bandwidth_limit_mb'] === 0 ? 'Unlimited' : number_format((float)$service['bandwidth_limit_mb'] / 1024, 1) . ' GB' ?></strong></div>
  59:                     <div><span>Domains</span><strong><?= (int)$service['domains_limit'] ?></strong></div>
  60:                     <div><span>Databases</span><strong><?= (int)$service['databases_limit'] ?></strong></div>
  61:                     <div><span>FTP Accounts</span><strong><?= (int)$service['ftp_accounts_limit'] ?></strong></div>
  62:                 </div>
  63:             </article>
  64:         <?php endforeach; ?>
  65:         </div>
  66:     <?php else: ?>
  67:         <div class="empty-state">
  68:             <div class="empty-icon">â—ˆ</div>
  69:             <h3>No services assigned</h3>
  70:             <p>There are currently no hosting services assigned to this client account.</p>
  71:             <a class="primary-button" href="<?= hl_url('plans.php') ?>">Browse hosting plans</a>
  72:         </div>
  73:     <?php endif; ?>
  74: </section>
  75:
  76: <?php require __DIR__ . '/includes/footer.php'; ?>

========================================
FILE: cpanel\includes\bootstrap.php
========================================
   1: <?php
   2: declare(strict_types=1);
   3:
   4: require_once __DIR__ . '/../../includes/public.php';
   5:
   6: $client = hl_require_client($pdo);
   7:
   8: if (!$client || ($client['id'] ?? 0) < 1) {
   9:     http_response_code(403);
  10:     exit('Access denied.');
  11: }

READ ONLY AUDIT COMPLETED
