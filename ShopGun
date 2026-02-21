<?php
// ShopGun - سیستم مدیریت فروشگاه جواهری
// نسخه 4.0 - رفع خطاها + شناسنامه محصول + بهبودهای اساسی

// تنظیمات اولیه
error_reporting(E_ALL);
ini_set('display_errors', 1);
session_start();
ob_start();

// تنظیمات دیتابیس
define('DB_HOST', 'localhost');
define('DB_NAME', 'mashahi8_dbjew');
define('DB_USER', 'mashahi8_userjew');
define('DB_PASS', '7U859cFcKh55hhBSavDW');
define('TABLE_PREFIX', 'wp_');

// کلاس مدیریت دیتابیس
class Database {
    private $pdo;
    
    public function __construct() {
        try {
            $this->pdo = new PDO("mysql:host=".DB_HOST.";dbname=".DB_NAME.";charset=utf8mb4", DB_USER, DB_PASS);
            $this->pdo->setAttribute(PDO::ATTR_ERRMODE, PDO::ERRMODE_EXCEPTION);
            $this->createTables();
            $this->insertSampleData();
        } catch(PDOException $e) {
            die("خطا در اتصال به دیتابیس: " . $e->getMessage());
        }
    }
    
    private function createTables() {
        $tables = [
            "products" => "CREATE TABLE IF NOT EXISTS ".TABLE_PREFIX."products (
                id INT AUTO_INCREMENT PRIMARY KEY,
                sku VARCHAR(100) UNIQUE NOT NULL,
                name VARCHAR(255) NOT NULL,
                price DECIMAL(10,2) NOT NULL,
                regular_price DECIMAL(10,2),
                sale_price DECIMAL(10,2),
                stock INT DEFAULT 0,
                category VARCHAR(100),
                categories TEXT,
                attributes TEXT,
                images TEXT,
                weight DECIMAL(10,2),
                length DECIMAL(10,2),
                width DECIMAL(10,2),
                height DECIMAL(10,2),
                description TEXT,
                created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
                updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
                INDEX idx_sku (sku),
                INDEX idx_category (category),
                INDEX idx_price (price),
                INDEX idx_stock (stock)
            )",
            
            "customers" => "CREATE TABLE IF NOT EXISTS ".TABLE_PREFIX."customers (
                id INT AUTO_INCREMENT PRIMARY KEY,
                fullname VARCHAR(255) NOT NULL,
                phone VARCHAR(20) UNIQUE NOT NULL,
                email VARCHAR(255),
                address TEXT,
                postal_code VARCHAR(20),
                city VARCHAR(100),
                province VARCHAR(100),
                total_orders INT DEFAULT 0,
                total_spent DECIMAL(10,2) DEFAULT 0,
                created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
                updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
                INDEX idx_phone (phone),
                INDEX idx_total_spent (total_spent)
            )",
            
            "orders" => "CREATE TABLE IF NOT EXISTS ".TABLE_PREFIX."orders (
                id INT AUTO_INCREMENT PRIMARY KEY,
                order_number VARCHAR(50) UNIQUE NOT NULL,
                customer_id INT,
                customer_phone VARCHAR(20),
                customer_name VARCHAR(255),
                total_amount DECIMAL(10,2) NOT NULL,
                status ENUM('pending', 'processing', 'completed', 'cancelled', 'stranger') DEFAULT 'pending',
                items TEXT,
                shipping_address TEXT,
                notes TEXT,
                created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
                updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
                FOREIGN KEY (customer_id) REFERENCES ".TABLE_PREFIX."customers(id),
                INDEX idx_order_number (order_number),
                INDEX idx_status (status),
                INDEX idx_created_at (created_at),
                INDEX idx_total_amount (total_amount)
            )",
            
            "settings" => "CREATE TABLE IF NOT EXISTS ".TABLE_PREFIX."settings (
                id INT AUTO_INCREMENT PRIMARY KEY,
                setting_key VARCHAR(100) UNIQUE NOT NULL,
                setting_value TEXT,
                created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
                updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP
            )",
            
            "logs" => "CREATE TABLE IF NOT EXISTS ".TABLE_PREFIX."logs (
                id INT AUTO_INCREMENT PRIMARY KEY,
                action VARCHAR(100) NOT NULL,
                description TEXT,
                user_id INT,
                user_name VARCHAR(255),
                ip_address VARCHAR(45),
                user_agent TEXT,
                created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
                INDEX idx_action (action),
                INDEX idx_created_at (created_at)
            )",
            
            "product_logs" => "CREATE TABLE IF NOT EXISTS ".TABLE_PREFIX."product_logs (
                id INT AUTO_INCREMENT PRIMARY KEY,
                product_id INT NOT NULL,
                product_name VARCHAR(255),
                action VARCHAR(100) NOT NULL,
                old_data TEXT,
                new_data TEXT,
                changed_fields TEXT,
                user_id INT,
                user_name VARCHAR(255),
                created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
                INDEX idx_product_id (product_id),
                INDEX idx_action (action),
                INDEX idx_created_at (created_at)
            )",

            // جدول جدید برای شناسنامه‌ها
            "certificates" => "CREATE TABLE IF NOT EXISTS ".TABLE_PREFIX."certificates (
                id INT AUTO_INCREMENT PRIMARY KEY,
                product_id INT,
                product_sku VARCHAR(100),
                product_name VARCHAR(255),
                product_data TEXT,
                certificate_number VARCHAR(50) UNIQUE NOT NULL,
                short_code VARCHAR(20) UNIQUE NOT NULL,
                issue_date DATE NOT NULL,
                expiry_date DATE,
                status ENUM('active', 'revoked', 'expired') DEFAULT 'active',
                attributes TEXT,
                image_url VARCHAR(500),
                qr_url VARCHAR(500),
                created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
                updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
                INDEX idx_certificate_number (certificate_number),
                INDEX idx_short_code (short_code),
                INDEX idx_status (status),
                INDEX idx_issue_date (issue_date)
            )"
        ];
        
        foreach($tables as $table => $sql) {
            try {
                $this->pdo->exec($sql);
            } catch(PDOException $e) {
                error_log("Error creating table: " . $e->getMessage());
            }
        }
    }
    
    private function insertSampleData() {
        // بررسی وجود داده‌های نمونه
        $productsCount = $this->fetch("SELECT COUNT(*) as count FROM ".TABLE_PREFIX."products")['count'];
        
        if ($productsCount == 0) {
            // درج محصولات نمونه
            $sampleProducts = [
                [
                    'sku' => 'GOLD-BRACELET-001',
                    'name' => 'دستبند طلا 18 عیار',
                    'price' => 2500000,
                    'regular_price' => 2800000,
                    'sale_price' => 2500000,
                    'stock' => 15,
                    'category' => 'دستبند',
                    'categories' => json_encode(['دستبند', 'طلا']),
                    'attributes' => json_encode([
                        ['name' => 'جنس', 'options' => ['طلا 18 عیار']],
                        ['name' => 'رنگ', 'options' => ['زرد', 'رزگلد']]
                    ]),
                    'images' => json_encode(['https://via.placeholder.com/400x400/FFD700/000000?text=دستبند+طلا']),
                    'weight' => 12.5,
                    'description' => 'دستبند طلای 18 عیار با طراحی مدرن'
                ],
                [
                    'sku' => 'SILVER-NECKLACE-001',
                    'name' => 'گردنبند نقره استرلینگ',
                    'price' => 450000,
                    'regular_price' => 520000,
                    'sale_price' => 450000,
                    'stock' => 8,
                    'category' => 'گردنبند',
                    'categories' => json_encode(['گردنبند', 'نقره']),
                    'attributes' => json_encode([
                        ['name' => 'جنس', 'options' => ['نقره استرلینگ']],
                        ['name' => 'طول', 'options' => ['45 سانتی‌متر']]
                    ]),
                    'images' => json_encode(['https://via.placeholder.com/400x400/C0C0C0/000000?text=گردنبند+نقره']),
                    'weight' => 8.2,
                    'description' => 'گردنبند نقره استرلینگ با نگین کریستال'
                ]
            ];
            
            foreach ($sampleProducts as $product) {
                $this->insert(TABLE_PREFIX.'products', $product);
            }
            
            // درج مشتریان نمونه
            $sampleCustomers = [
                [
                    'fullname' => 'سمیه سعیدیان',
                    'phone' => '09123456789',
                    'email' => 'somiyeh@example.com',
                    'address' => 'میدان آزادگان بلوار شهید عابدی خیابان باقرالعلوم کوچه ۸ پلاک ۷',
                    'postal_code' => '3718898575',
                    'city' => 'تهران',
                    'province' => 'تهران',
                    'total_orders' => 3,
                    'total_spent' => 7850000
                ],
                [
                    'fullname' => 'محمد رضایی',
                    'phone' => '09129876543',
                    'email' => 'mohammad@example.com',
                    'address' => 'خیابان ولیعصر، پلاک ۱۲۳',
                    'postal_code' => '1234567890',
                    'city' => 'تهران',
                    'province' => 'تهران',
                    'total_orders' => 2,
                    'total_spent' => 3200000
                ]
            ];
            
            foreach ($sampleCustomers as $customer) {
                $this->insert(TABLE_PREFIX.'customers', $customer);
            }
            
            // درج سفارشات نمونه
            $sampleOrders = [
                [
                    'order_number' => 'SG-20241115-001',
                    'customer_id' => 1,
                    'customer_phone' => '09123456789',
                    'customer_name' => 'سمیه سعیدیان',
                    'total_amount' => 2500000,
                    'status' => 'completed',
                    'items' => json_encode([
                        ['product_id' => 1, 'name' => 'دستبند طلا 18 عیار', 'quantity' => 1, 'price' => 2500000]
                    ]),
                    'shipping_address' => json_encode([
                        'address_1' => 'میدان آزادگان بلوار شهید عابدی خیابان باقرالعلوم کوچه ۸ پلاک ۷',
                        'city' => 'تهران',
                        'state' => 'تهران',
                        'postcode' => '3718898575'
                    ])
                ]
            ];
            
            foreach ($sampleOrders as $order) {
                $this->insert(TABLE_PREFIX.'orders', $order);
            }
            
            // درج تنظیمات نمونه
            $sampleSettings = [
                ['setting_key' => 'store_name', 'setting_value' => 'فروشگاه جواهری شاپگان'],
                ['setting_key' => 'store_url', 'setting_value' => 'https://mashahir.jewelry'],
                ['setting_key' => 'primary_color', 'setting_value' => '#40E0D0'],
                ['setting_key' => 'theme', 'setting_value' => 'dark']
            ];
            
            foreach ($sampleSettings as $setting) {
                $this->insert(TABLE_PREFIX.'settings', $setting);
            }
        }
    }
    
    public function query($sql, $params = []) {
        $stmt = $this->pdo->prepare($sql);
        $stmt->execute($params);
        return $stmt;
    }
    
    public function fetchAll($sql, $params = []) {
        $stmt = $this->query($sql, $params);
        return $stmt->fetchAll(PDO::FETCH_ASSOC);
    }
    
    public function fetch($sql, $params = []) {
        $stmt = $this->query($sql, $params);
        return $stmt->fetch(PDO::FETCH_ASSOC);
    }
    
    public function insert($table, $data) {
        $columns = implode(', ', array_keys($data));
        $placeholders = ':' . implode(', :', array_keys($data));
        $sql = "INSERT INTO $table ($columns) VALUES ($placeholders)";
        $this->query($sql, $data);
        return $this->pdo->lastInsertId();
    }
    
    public function update($table, $data, $where) {
        $set = [];
        foreach($data as $key => $value) {
            $set[] = "$key = :$key";
        }
        $setStr = implode(', ', $set);
        $sql = "UPDATE $table SET $setStr WHERE $where";
        return $this->query($sql, $data)->rowCount();
    }
    
    public function delete($table, $where, $params = []) {
        $sql = "DELETE FROM $table WHERE $where";
        return $this->query($sql, $params)->rowCount();
    }

    // تابع جدید برای رفع خطای SKU تکراری
    public function fixDuplicateSKUs() {
        try {
            // پیدا کردن SKU های تکراری
            $sql = "SELECT sku, COUNT(*) as count, GROUP_CONCAT(id) as ids 
                    FROM ".TABLE_PREFIX."products 
                    GROUP BY sku 
                    HAVING count > 1";
            
            $duplicates = $this->fetchAll($sql);
            $fixed = 0;
            
            foreach($duplicates as $dup) {
                $ids = explode(',', $dup['ids']);
                $keep_id = $ids[0]; // نگه داشتن اولین رکورد
                
                // حذف رکوردهای تکراری (به جز اولین)
                for ($i = 1; $i < count($ids); $i++) {
                    $this->delete(TABLE_PREFIX.'products', "id = ?", [$ids[$i]]);
                    $fixed++;
                }
            }
            
            return ['success' => true, 'fixed' => $fixed, 'message' => "{$fixed} رکورد تکراری حذف شد"];
            
        } catch (PDOException $e) {
            return ['success' => false, 'message' => 'خطا در رفع مشکل: ' . $e->getMessage()];
        }
    }

    // تابع جدید برای ذخیره شناسنامه
    public function saveCertificate($data) {
        return $this->insert(TABLE_PREFIX.'certificates', $data);
    }

    // تابع جدید برای دریافت شناسنامه‌ها
    public function getCertificates($filters = [], $page = 1, $per_page = 25) {
        $where = [];
        $params = [];
        $offset = ($page - 1) * $per_page;

        if(!empty($filters['search'])) {
            $where[] = "(certificate_number LIKE ? OR product_name LIKE ? OR short_code LIKE ?)";
            $search = '%' . $filters['search'] . '%';
            $params[] = $search;
            $params[] = $search;
            $params[] = $search;
        }

        if(!empty($filters['status'])) {
            $where[] = "status = ?";
            $params[] = $filters['status'];
        }

        if(!empty($filters['date_from'])) {
            $where[] = "issue_date >= ?";
            $params[] = $filters['date_from'];
        }

        if(!empty($filters['date_to'])) {
            $where[] = "issue_date <= ?";
            $params[] = $filters['date_to'];
        }

        $whereClause = $where ? "WHERE " . implode(" AND ", $where) : "";

        $certificates = $this->fetchAll(
            "SELECT * FROM ".TABLE_PREFIX."certificates $whereClause ORDER BY issue_date DESC LIMIT $offset, $per_page",
            $params
        );

        $total = $this->fetch(
            "SELECT COUNT(*) as total FROM ".TABLE_PREFIX."certificates $whereClause",
            $params
        )['total'];

        // پردازش داده‌های JSON
        foreach($certificates as &$cert) {
            $cert['product_data'] = json_decode($cert['product_data'], true) ?: [];
            $cert['attributes'] = json_decode($cert['attributes'], true) ?: [];
        }

        return [
            'success' => true,
            'certificates' => $certificates,
            'pagination' => [
                'page' => $page,
                'per_page' => $per_page,
                'total' => $total,
                'total_pages' => ceil($total / $per_page)
            ]
        ];
    }
}

// کلاس مدیریت WooCommerce
class WooCommerceManager {
    private $consumerKey;
    private $consumerSecret;
    private $storeURL;
    private $db;
    
    public function __construct($db) {
        $this->db = $db;
        $this->loadSettings();
    }
    
    private function loadSettings() {
        $settings = $this->db->fetchAll("SELECT setting_key, setting_value FROM ".TABLE_PREFIX."settings");
        foreach($settings as $setting) {
            if($setting['setting_key'] == 'woocommerce_consumer_key') {
                $this->consumerKey = $setting['setting_value'];
            } elseif($setting['setting_key'] == 'woocommerce_consumer_secret') {
                $this->consumerSecret = $setting['setting_value'];
            } elseif($setting['setting_key'] == 'woocommerce_store_url') {
                $this->storeURL = $setting['setting_value'];
            }
        }
    }
    
    public function testConnection() {
        if(!$this->consumerKey || !$this->consumerSecret || !$this->storeURL) {
            return ['success' => false, 'message' => 'تنظیمات WooCommerce کامل نیست'];
        }
        
        try {
            $url = rtrim($this->storeURL, '/') . '/wp-json/wc/v3/products?per_page=1';
            $response = $this->makeRequest($url);
            return ['success' => true, 'message' => 'اتصال موفقیت‌آمیز بود'];
        } catch(Exception $e) {
            return ['success' => false, 'message' => $e->getMessage()];
        }
    }
    
    public function syncProducts($page = 1, $per_page = 100) {
        if(!$this->consumerKey || !$this->consumerSecret || !$this->storeURL) {
            throw new Exception('تنظیمات WooCommerce کامل نیست');
        }
        
        $url = rtrim($this->storeURL, '/') . "/wp-json/wc/v3/products?page=$page&per_page=$per_page";
        $products = $this->makeRequest($url);
        
        $synced = 0;
        $errors = [];
        
        foreach($products as $product) {
            try {
                $sku = !empty($product['sku']) ? $product['sku'] : 'WC-' . $product['id'];
                
                $existing = $this->db->fetch(
                    "SELECT id FROM ".TABLE_PREFIX."products WHERE sku = ?", 
                    [$sku]
                );
                
                $categories = [];
                if(!empty($product['categories'])) {
                    foreach($product['categories'] as $category) {
                        $categories[] = $category['name'];
                    }
                }
                
                $attributes = [];
                if(!empty($product['attributes'])) {
                    foreach($product['attributes'] as $attribute) {
                        $attributes[] = [
                            'name' => $attribute['name'],
                            'options' => $attribute['options']
                        ];
                    }
                }
                
                $images = [];
                if(!empty($product['images'])) {
                    foreach($product['images'] as $image) {
                        $images[] = $image['src'];
                    }
                }
                
                $productData = [
                    'sku' => $sku,
                    'name' => $product['name'],
                    'price' => $product['price'],
                    'regular_price' => $product['regular_price'],
                    'sale_price' => $product['sale_price'],
                    'stock' => $product['stock_quantity'] ?: 0,
                    'category' => !empty($product['categories']) ? $product['categories'][0]['name'] : 'دسته‌بندی نشده',
                    'categories' => json_encode($categories, JSON_UNESCAPED_UNICODE),
                    'attributes' => json_encode($attributes, JSON_UNESCAPED_UNICODE),
                    'images' => json_encode($images, JSON_UNESCAPED_UNICODE),
                    'weight' => $product['weight'] ?: 0,
                    'length' => $product['dimensions']['length'] ?: 0,
                    'width' => $product['dimensions']['width'] ?: 0,
                    'height' => $product['dimensions']['height'] ?: 0,
                    'description' => $product['description']
                ];
                
                if($existing) {
                    $this->db->update(TABLE_PREFIX.'products', $productData, "id = " . $existing['id']);
                    
                    $this->db->insert(TABLE_PREFIX.'product_logs', [
                        'product_id' => $existing['id'],
                        'product_name' => $product['name'],
                        'action' => 'update_sync',
                        'changed_fields' => json_encode(array_keys($productData)),
                        'user_name' => 'system_sync'
                    ]);
                    
                    $synced++;
                } else {
                    try {
                        $productId = $this->db->insert(TABLE_PREFIX.'products', $productData);
                        
                        $this->db->insert(TABLE_PREFIX.'product_logs', [
                            'product_id' => $productId,
                            'product_name' => $product['name'],
                            'action' => 'create_sync',
                            'user_name' => 'system_sync'
                        ]);
                        
                        $synced++;
                    } catch (PDOException $e) {
                        if (strpos($e->getMessage(), '1062 Duplicate entry') !== false) {
                            $new_sku = $sku . '-' . time() . '-' . rand(100, 999);
                            $productData['sku'] = $new_sku;
                            
                            try {
                                $productId = $this->db->insert(TABLE_PREFIX.'products', $productData);
                                
                                $this->db->insert(TABLE_PREFIX.'product_logs', [
                                    'product_id' => $productId,
                                    'product_name' => $product['name'],
                                    'action' => 'create_sync_duplicate_fixed',
                                    'user_name' => 'system_sync'
                                ]);
                                
                                $synced++;
                                $errors[] = "SKU تکراری حل شد: {$sku} -> {$new_sku}";
                            } catch (PDOException $e2) {
                                $errors[] = "خطا در درج محصول {$product['name']} با SKU جدید: " . $e2->getMessage();
                            }
                        } else {
                            $errors[] = "خطا در درج محصول {$product['name']}: " . $e->getMessage();
                        }
                    }
                }
            } catch (Exception $e) {
                $errors[] = "خطا در پردازش محصول {$product['name']}: " . $e->getMessage();
            }
        }
        
        return [
            'synced' => $synced, 
            'has_more' => count($products) == $per_page,
            'errors' => $errors
        ];
    }
    
    public function syncOrders($page = 1, $per_page = 100) {
        if(!$this->consumerKey || !$this->consumerSecret || !$this->storeURL) {
            throw new Exception('تنظیمات WooCommerce کامل نیست');
        }
        
        $url = rtrim($this->storeURL, '/') . "/wp-json/wc/v3/orders?page=$page&per_page=$per_page";
        $orders = $this->makeRequest($url);
        
        $synced = 0;
        foreach($orders as $order) {
            $customerId = null;
            if($order['billing']['phone']) {
                $existingCustomer = $this->db->fetch(
                    "SELECT id FROM ".TABLE_PREFIX."customers WHERE phone = ?", 
                    [$order['billing']['phone']]
                );
                
                if(!$existingCustomer) {
                    $customerId = $this->db->insert(TABLE_PREFIX.'customers', [
                        'fullname' => $order['billing']['first_name'] . ' ' . $order['billing']['last_name'],
                        'phone' => $order['billing']['phone'],
                        'email' => $order['billing']['email'],
                        'address' => $order['billing']['address_1'],
                        'postal_code' => $order['billing']['postcode'],
                        'city' => $order['billing']['city'],
                        'province' => $order['billing']['state']
                    ]);
                } else {
                    $customerId = $existingCustomer['id'];
                }
            }
            
            $items = [];
            foreach($order['line_items'] as $item) {
                $items[] = [
                    'product_id' => $item['product_id'],
                    'name' => $item['name'],
                    'quantity' => $item['quantity'],
                    'price' => $item['price']
                ];
            }
            
            $orderData = [
                'order_number' => $order['number'],
                'customer_id' => $customerId,
                'customer_phone' => $order['billing']['phone'],
                'customer_name' => $order['billing']['first_name'] . ' ' . $order['billing']['last_name'],
                'total_amount' => $order['total'],
                'status' => $order['status'],
                'items' => json_encode($items, JSON_UNESCAPED_UNICODE),
                'shipping_address' => json_encode($order['billing'], JSON_UNESCAPED_UNICODE),
                'notes' => $order['customer_note']
            ];
            
            $existingOrder = $this->db->fetch(
                "SELECT id FROM ".TABLE_PREFIX."orders WHERE order_number = ?", 
                [$order['number']]
            );
            
            if($existingOrder) {
                $this->db->update(TABLE_PREFIX.'orders', $orderData, "id = " . $existingOrder['id']);
            } else {
                $this->db->insert(TABLE_PREFIX.'orders', $orderData);
            }
            $synced++;
        }
        
        return ['synced' => $synced, 'has_more' => count($orders) == $per_page];
    }
    
    private function makeRequest($url) {
        $ch = curl_init();
        curl_setopt_array($ch, [
            CURLOPT_URL => $url,
            CURLOPT_RETURNTRANSFER => true,
            CURLOPT_USERPWD => $this->consumerKey . ':' . $this->consumerSecret,
            CURLOPT_HTTPAUTH => CURLAUTH_BASIC,
            CURLOPT_TIMEOUT => 30,
            CURLOPT_SSL_VERIFYPEER => false,
            CURLOPT_USERAGENT => 'ShopGun/4.0'
        ]);
        
        $response = curl_exec($ch);
        $httpCode = curl_getinfo($ch, CURLINFO_HTTP_CODE);
        $error = curl_error($ch);
        curl_close($ch);
        
        if($error) {
            throw new Exception('خطای شبکه: ' . $error);
        }
        
        if($httpCode != 200) {
            throw new Exception('خطای HTTP: ' . $httpCode);
        }
        
        $data = json_decode($response, true);
        if(json_last_error() !== JSON_ERROR_NONE) {
            throw new Exception('خطای پردازش JSON');
        }
        
        return $data;
    }
}

// کلاس مدیریت API داخلی
class ShopGunAPI {
    private $db;
    private $woo;
    
    public function __construct($db, $woo) {
        $this->db = $db;
        $this->woo = $woo;
    }
    
    public function handleRequest() {
        $action = $_POST['action'] ?? $_GET['action'] ?? '';
        
        switch($action) {
            case 'get_products':
                return $this->getProducts();
            case 'get_product':
                return $this->getProduct();
            case 'update_product':
                return $this->updateProduct();
            case 'search_product_by_sku':
                return $this->searchProductBySku();
            case 'bulk_price_update_percent':
                return $this->bulkPriceUpdatePercent();
            case 'get_orders':
                return $this->getOrders();
            case 'get_order':
                return $this->getOrder();
            case 'create_stranger_order':
                return $this->createStrangerOrder();
            case 'search_customer_by_phone':
                return $this->searchCustomerByPhone();
            case 'get_comprehensive_stats':
                return $this->getComprehensiveStats();
            case 'get_sales_comparison':
                return $this->getSalesComparison();
            case 'save_settings':
                return $this->saveSettings();
            case 'delete_settings':
                return $this->deleteSettings();
            case 'test_connection':
                return $this->testConnection();
            case 'sync_products':
                return $this->syncProducts();
            case 'sync_orders':
                return $this->syncOrders();
            case 'get_categories':
                return $this->getCategories();
            case 'get_attributes':
                return $this->getAttributes();
            case 'create_customer_order':
                return $this->createCustomerOrder();
            case 'get_logs':
                return $this->getLogs();
            case 'get_product_logs':
                return $this->getProductLogs();
            case 'fix_duplicate_skus':
                return $this->fixDuplicateSKUs();
            // توابع جدید برای شناسنامه
            case 'search_products_for_certificate':
                return $this->searchProductsForCertificate();
            case 'get_product_attributes':
                return $this->getProductAttributes();
            case 'issue_certificate':
                return $this->issueCertificate();
            case 'get_certificates':
                return $this->getCertificates();
            case 'get_certificate':
                return $this->getCertificate();
            case 'revoke_certificate':
                return $this->revokeCertificate();
            case 'verify_certificate':
                return $this->verifyCertificate();
            default:
                return ['success' => false, 'message' => 'Action not found'];
        }
    }
    
    private function getProducts() {
        $page = intval($_GET['page'] ?? 1);
        $per_page = intval($_GET['per_page'] ?? 50);
        $search = $_GET['search'] ?? '';
        $category = $_GET['category'] ?? '';
        $attribute = $_GET['attribute'] ?? '';
        $sort = $_GET['sort'] ?? 'id';
        $order = $_GET['order'] ?? 'DESC';
        
        $offset = ($page - 1) * $per_page;
        $where = [];
        $params = [];
        
        if($search) {
            $where[] = "(name LIKE ? OR sku LIKE ?)";
            $params[] = "%$search%";
            $params[] = "%$search%";
        }
        
        if($category) {
            $where[] = "(category = ? OR categories LIKE ?)";
            $params[] = $category;
            $params[] = "%$category%";
        }
        
        if($attribute) {
            $where[] = "attributes LIKE ?";
            $params[] = "%$attribute%";
        }
        
        $whereClause = $where ? "WHERE " . implode(" AND ", $where) : "";
        
        $allowedSort = ['id', 'name', 'sku', 'price', 'stock', 'category'];
        if(!in_array($sort, $allowedSort)) $sort = 'id';
        $order = strtoupper($order) == 'ASC' ? 'ASC' : 'DESC';
        
        $products = $this->db->fetchAll(
            "SELECT * FROM ".TABLE_PREFIX."products $whereClause ORDER BY $sort $order LIMIT $offset, $per_page",
            $params
        );
        
        $total = $this->db->fetch(
            "SELECT COUNT(*) as total FROM ".TABLE_PREFIX."products $whereClause",
            $params
        )['total'];
        
        foreach($products as &$product) {
            $product['categories'] = json_decode($product['categories'], true) ?: [];
            $product['attributes'] = json_decode($product['attributes'], true) ?: [];
            $product['images'] = json_decode($product['images'], true) ?: [];
            $product['price_per_gram'] = $product['weight'] > 0 ? round($product['price'] / $product['weight'], 2) : 0;
        }
        
        return [
            'success' => true,
            'products' => $products,
            'pagination' => [
                'page' => $page,
                'per_page' => $per_page,
                'total' => $total,
                'total_pages' => ceil($total / $per_page)
            ]
        ];
    }
    
    private function getProduct() {
        $id = intval($_GET['id'] ?? 0);
        $product = $this->db->fetch("SELECT * FROM ".TABLE_PREFIX."products WHERE id = ?", [$id]);
        
        if($product) {
            $product['categories'] = json_decode($product['categories'], true) ?: [];
            $product['attributes'] = json_decode($product['attributes'], true) ?: [];
            $product['images'] = json_decode($product['images'], true) ?: [];
            return ['success' => true, 'product' => $product];
        } else {
            return ['success' => false, 'message' => 'محصول یافت نشد'];
        }
    }
    
    private function updateProduct() {
        $id = intval($_POST['id'] ?? 0);
        $name = trim($_POST['name'] ?? '');
        $price = floatval($_POST['price'] ?? 0);
        $regular_price = floatval($_POST['regular_price'] ?? 0);
        $sale_price = floatval($_POST['sale_price'] ?? 0);
        $stock = intval($_POST['stock'] ?? 0);
        $weight = floatval($_POST['weight'] ?? 0);
        $length = floatval($_POST['length'] ?? 0);
        $width = floatval($_POST['width'] ?? 0);
        $height = floatval($_POST['height'] ?? 0);
        
        if(!$id || !$name || $price < 0) {
            return ['success' => false, 'message' => 'داده‌های نامعتبر'];
        }
        
        $oldProduct = $this->db->fetch("SELECT * FROM ".TABLE_PREFIX."products WHERE id = ?", [$id]);
        
        $updated = $this->db->update(TABLE_PREFIX.'products', [
            'name' => $name,
            'price' => $price,
            'regular_price' => $regular_price,
            'sale_price' => $sale_price,
            'stock' => $stock,
            'weight' => $weight,
            'length' => $length,
            'width' => $width,
            'height' => $height
        ], "id = $id");
        
        if($updated > 0) {
            $changedFields = [];
            $oldData = [];
            $newData = [];
            
            $fields = ['name', 'price', 'regular_price', 'sale_price', 'stock', 'weight', 'length', 'width', 'height'];
            foreach($fields as $field) {
                if($oldProduct[$field] != ${$field}) {
                    $changedFields[] = $field;
                    $oldData[$field] = $oldProduct[$field];
                    $newData[$field] = ${$field};
                }
            }
            
            if(!empty($changedFields)) {
                $this->db->insert(TABLE_PREFIX.'product_logs', [
                    'product_id' => $id,
                    'product_name' => $name,
                    'action' => 'update',
                    'old_data' => json_encode($oldData, JSON_UNESCAPED_UNICODE),
                    'new_data' => json_encode($newData, JSON_UNESCAPED_UNICODE),
                    'changed_fields' => json_encode($changedFields, JSON_UNESCAPED_UNICODE),
                    'user_name' => 'admin'
                ]);
            }
        }
        
        return ['success' => $updated > 0, 'message' => $updated > 0 ? 'محصول به‌روزرسانی شد' : 'خطا در به‌روزرسانی'];
    }
    
    private function searchProductBySku() {
        $sku = $_GET['sku'] ?? '';
        $products = $this->db->fetchAll("SELECT * FROM ".TABLE_PREFIX."products WHERE sku LIKE ? LIMIT 10", ["%$sku%"]);
        
        foreach($products as &$product) {
            $product['images'] = json_decode($product['images'], true) ?: [];
        }
        
        return ['success' => true, 'products' => $products];
    }
    
    private function bulkPriceUpdatePercent() {
        $category = $_POST['category'] ?? '';
        $attribute = $_POST['attribute'] ?? '';
        $percent = floatval($_POST['percent'] ?? 0);
        $limit = intval($_POST['limit'] ?? 0);
        
        if($percent == 0) {
            return ['success' => false, 'message' => 'درصد تغییر الزامی است'];
        }
        
        $where = [];
        $params = [];
        
        if($category) {
            $where[] = "(category = ? OR categories LIKE ?)";
            $params[] = $category;
            $params[] = "%$category%";
        }
        
        if($attribute) {
            $where[] = "attributes LIKE ?";
            $params[] = "%$attribute%";
        }
        
        $whereClause = $where ? "WHERE " . implode(" AND ", $where) : "";
        $limitClause = $limit ? "LIMIT $limit" : "";
        
        $products = $this->db->fetchAll("SELECT id, name, price FROM ".TABLE_PREFIX."products $whereClause $limitClause", $params);
        
        $operator = $percent > 0 ? '*' : '/';
        $factor = $percent > 0 ? (1 + $percent/100) : (1 - abs($percent)/100);
        
        $updated = $this->db->query(
            "UPDATE ".TABLE_PREFIX."products SET price = price $operator ?, regular_price = regular_price $operator ? $whereClause $limitClause",
            array_merge([$factor, $factor], $params)
        )->rowCount();
        
        foreach($products as $product) {
            $newPrice = $operator == '*' ? $product['price'] * $factor : $product['price'] / $factor;
            
            $this->db->insert(TABLE_PREFIX.'product_logs', [
                'product_id' => $product['id'],
                'product_name' => $product['name'],
                'action' => 'bulk_price_update',
                'old_data' => json_encode(['price' => $product['price']], JSON_UNESCAPED_UNICODE),
                'new_data' => json_encode(['price' => $newPrice], JSON_UNESCAPED_UNICODE),
                'changed_fields' => json_encode(['price'], JSON_UNESCAPED_UNICODE),
                'user_name' => 'admin'
            ]);
        }
        
        return ['success' => true, 'updated' => $updated, 'message' => "$updated محصول به‌روزرسانی شد"];
    }
    
    private function getOrders() {
        $page = intval($_GET['page'] ?? 1);
        $per_page = intval($_GET['per_page'] ?? 50);
        $status = $_GET['status'] ?? '';
        
        $offset = ($page - 1) * $per_page;
        $where = [];
        $params = [];
        
        if($status) {
            $where[] = "status = ?";
            $params[] = $status;
        }
        
        $whereClause = $where ? "WHERE " . implode(" AND ", $where) : "";
        
        $orders = $this->db->fetchAll(
            "SELECT o.*, c.fullname, c.phone FROM ".TABLE_PREFIX."orders o 
             LEFT JOIN ".TABLE_PREFIX."customers c ON o.customer_id = c.id 
             $whereClause ORDER BY o.created_at DESC LIMIT $offset, $per_page",
            $params
        );
        
        $total = $this->db->fetch(
            "SELECT COUNT(*) as total FROM ".TABLE_PREFIX."orders o $whereClause",
            $params
        )['total'];
        
        foreach($orders as &$order) {
            $order['items'] = json_decode($order['items'], true) ?: [];
            $order['shipping_address'] = json_decode($order['shipping_address'], true) ?: [];
        }
        
        return [
            'success' => true,
            'orders' => $orders,
            'pagination' => [
                'page' => $page,
                'per_page' => $per_page,
                'total' => $total,
                'total_pages' => ceil($total / $per_page)
            ]
        ];
    }
    
    private function getOrder() {
        $id = intval($_GET['id'] ?? 0);
        $order = $this->db->fetch(
            "SELECT o.*, c.fullname, c.phone, c.address, c.postal_code, c.city, c.province 
             FROM ".TABLE_PREFIX."orders o 
             LEFT JOIN ".TABLE_PREFIX."customers c ON o.customer_id = c.id 
             WHERE o.id = ?", 
            [$id]
        );
        
        if($order) {
            $order['items'] = json_decode($order['items'], true) ?: [];
            $order['shipping_address'] = json_decode($order['shipping_address'], true) ?: [];
            return ['success' => true, 'order' => $order];
        } else {
            return ['success' => false, 'message' => 'سفارش یافت نشد'];
        }
    }
    
    private function createStrangerOrder() {
        return $this->createCustomerOrder('stranger');
    }
    
    private function createCustomerOrder($type = 'stranger') {
        $customerPhone = $_POST['customer_phone'] ?? '';
        $customerName = $_POST['customer_name'] ?? 'مشتری ناشناس';
        $customerAddress = $_POST['customer_address'] ?? '';
        $customerPostal = $_POST['customer_postal'] ?? '';
        $customerCity = $_POST['customer_city'] ?? '';
        $customerProvince = $_POST['customer_province'] ?? '';
        $notes = $_POST['notes'] ?? '';
        $items = $_POST['items'] ?? [];
        $total = floatval($_POST['total'] ?? 0);
        
        if(empty($items) || $total <= 0) {
            return ['success' => false, 'message' => 'داده‌های سفارش نامعتبر'];
        }
        
        $customerId = null;
        if($customerPhone) {
            $existingCustomer = $this->db->fetch(
                "SELECT id FROM ".TABLE_PREFIX."customers WHERE phone = ?", 
                [$customerPhone]
            );
            
            if($existingCustomer) {
                $customerId = $existingCustomer['id'];
                $this->db->update(TABLE_PREFIX.'customers', [
                    'fullname' => $customerName,
                    'address' => $customerAddress,
                    'postal_code' => $customerPostal,
                    'city' => $customerCity,
                    'province' => $customerProvince
                ], "id = $customerId");
            } else {
                $customerId = $this->db->insert(TABLE_PREFIX.'customers', [
                    'fullname' => $customerName,
                    'phone' => $customerPhone,
                    'address' => $customerAddress,
                    'postal_code' => $customerPostal,
                    'city' => $customerCity,
                    'province' => $customerProvince
                ]);
            }
        }
        
        $processedItems = [];
        foreach($items as $item) {
            if(isset($item['id']) && $item['id'] > 0) {
                $product = $this->db->fetch("SELECT sku, name FROM ".TABLE_PREFIX."products WHERE id = ?", [$item['id']]);
                if($product) {
                    $processedItems[] = [
                        'product_id' => $item['id'],
                        'sku' => $product['sku'],
                        'name' => $product['name'],
                        'quantity' => $item['quantity'],
                        'price' => $item['price']
                    ];
                } else {
                    $processedItems[] = [
                        'product_id' => 0,
                        'sku' => $item['sku'] ?? '',
                        'name' => $item['name'] ?? 'نامشخص',
                        'quantity' => $item['quantity'],
                        'price' => $item['price']
                    ];
                }
            } else {
                $processedItems[] = [
                    'product_id' => 0,
                    'sku' => $item['sku'] ?? '',
                    'name' => $item['name'] ?? 'محصول خارج از بانک',
                    'quantity' => $item['quantity'],
                    'price' => $item['price']
                ];
            }
        }
        
        $orderNumber = 'SG-' . date('Ymd-His') . '-' . rand(1000, 9999);
        $orderId = $this->db->insert(TABLE_PREFIX.'orders', [
            'order_number' => $orderNumber,
            'customer_id' => $customerId,
            'customer_phone' => $customerPhone,
            'customer_name' => $customerName,
            'total_amount' => $total,
            'status' => $type == 'stranger' ? 'stranger' : 'pending',
            'items' => json_encode($processedItems, JSON_UNESCAPED_UNICODE),
            'shipping_address' => json_encode([
                'address_1' => $customerAddress,
                'city' => $customerCity,
                'state' => $customerProvince,
                'postcode' => $customerPostal
            ], JSON_UNESCAPED_UNICODE),
            'notes' => $notes
        ]);
        
        $this->db->insert(TABLE_PREFIX.'logs', [
            'action' => 'create_order',
            'description' => "سفارش جدید با شماره $orderNumber ایجاد شد",
            'user_name' => 'admin'
        ]);
        
        return [
            'success' => true, 
            'order_id' => $orderId,
            'order_number' => $orderNumber,
            'message' => 'سفارش با موفقیت ثبت شد'
        ];
    }
    private function generateUniqueShortCode() {
    do {
        $code = substr(md5(uniqid(mt_rand(), true)), 0, 10); // 10 کاراکتر
        $exists = $this->db->fetch("SELECT id FROM ".TABLE_PREFIX."certificates WHERE short_code = ?", [$code]);
    } while($exists);
    return $code;
}
    private function searchCustomerByPhone() {
        $phone = $_GET['phone'] ?? '';
        $customer = $this->db->fetch("SELECT * FROM ".TABLE_PREFIX."customers WHERE phone LIKE ?", ["%$phone%"]);
        
        if($customer) {
            return ['success' => true, 'customer' => $customer];
        } else {
            return ['success' => false, 'message' => 'مشتری یافت نشد'];
        }
    }
    
    private function getComprehensiveStats() {
        $totalProducts = $this->db->fetch("SELECT COUNT(*) as total FROM ".TABLE_PREFIX."products")['total'];
        $totalProductsValue = $this->db->fetch("SELECT SUM(price * stock) as total FROM ".TABLE_PREFIX."products WHERE stock > 0")['total'] ?: 0;
        $totalCustomers = $this->db->fetch("SELECT COUNT(*) as total FROM ".TABLE_PREFIX."customers")['total'];
        $customersWithOrders = $this->db->fetch("SELECT COUNT(DISTINCT customer_id) as total FROM ".TABLE_PREFIX."orders WHERE customer_id IS NOT NULL")['total'];
        $totalOrders = $this->db->fetch("SELECT COUNT(*) as total FROM ".TABLE_PREFIX."orders")['total'];
        $totalRevenue = $this->db->fetch("SELECT SUM(total_amount) as total FROM ".TABLE_PREFIX."orders WHERE status = 'completed'")['total'] ?: 0;
        
        $dailySales = $this->db->fetch("SELECT SUM(total_amount) as total FROM ".TABLE_PREFIX."orders WHERE DATE(created_at) = CURDATE() AND status = 'completed'")['total'] ?: 0;
        $weeklySales = $this->db->fetch("SELECT SUM(total_amount) as total FROM ".TABLE_PREFIX."orders WHERE YEARWEEK(created_at) = YEARWEEK(CURDATE()) AND status = 'completed'")['total'] ?: 0;
        $monthlySales = $this->db->fetch("SELECT SUM(total_amount) as total FROM ".TABLE_PREFIX."orders WHERE YEAR(created_at) = YEAR(CURDATE()) AND MONTH(created_at) = MONTH(CURDATE()) AND status = 'completed'")['total'] ?: 0;
        $yearlySales = $this->db->fetch("SELECT SUM(total_amount) as total FROM ".TABLE_PREFIX."orders WHERE YEAR(created_at) = YEAR(CURDATE()) AND status = 'completed'")['total'] ?: 0;
        
        $orders = $this->db->fetchAll("SELECT items FROM ".TABLE_PREFIX."orders WHERE status = 'completed'");
        $productCounts = [];
        foreach ($orders as $order) {
            $items = json_decode($order['items'], true);
            if (is_array($items)) {
                foreach ($items as $item) {
                    $pid = $item['product_id'] ?? 0;
                    $qty = $item['quantity'] ?? 1;
                    if ($pid) {
                        if (!isset($productCounts[$pid])) {
                            $productCounts[$pid] = 0;
                        }
                        $productCounts[$pid] += $qty;
                    }
                }
            }
        }
        arsort($productCounts);
        $topProducts = [];
        $counter = 0;
        foreach ($productCounts as $pid => $totalQty) {
            if ($counter >= 10) break;
            $product = $this->db->fetch("SELECT name, sku, stock FROM ".TABLE_PREFIX."products WHERE id = ?", [$pid]);
            if ($product) {
                $product['total_quantity'] = $totalQty;
                $topProducts[] = $product;
                $counter++;
            }
        }
        
        $topCustomers = $this->db->fetchAll("
            SELECT c.fullname, c.phone, COUNT(o.id) as order_count, SUM(o.total_amount) as total_spent 
            FROM ".TABLE_PREFIX."customers c 
            JOIN ".TABLE_PREFIX."orders o ON c.id = o.customer_id 
            WHERE o.status = 'completed'
            GROUP BY c.id 
            ORDER BY total_spent DESC 
            LIMIT 10
        ");
        
        $lowStock = $this->db->fetch("SELECT COUNT(*) as total FROM ".TABLE_PREFIX."products WHERE stock < 10 AND stock > 0")['total'];
        $outOfStock = $this->db->fetch("SELECT COUNT(*) as total FROM ".TABLE_PREFIX."products WHERE stock = 0")['total'];
        
        // آمار شناسنامه‌ها
        $totalCertificates = $this->db->fetch("SELECT COUNT(*) as total FROM ".TABLE_PREFIX."certificates")['total'] ?: 0;
        $todayCertificates = $this->db->fetch("SELECT COUNT(*) as total FROM ".TABLE_PREFIX."certificates WHERE DATE(issue_date) = CURDATE()")['total'] ?: 0;
        $activeCertificates = $this->db->fetch("SELECT COUNT(*) as total FROM ".TABLE_PREFIX."certificates WHERE status = 'active'")['total'] ?: 0;
        $revokedCertificates = $this->db->fetch("SELECT COUNT(*) as total FROM ".TABLE_PREFIX."certificates WHERE status = 'revoked'")['total'] ?: 0;
        
        return [
            'success' => true,
            'stats' => [
                'total_products' => $totalProducts,
                'total_products_value' => $totalProductsValue,
                'total_customers' => $totalCustomers,
                'customers_with_orders' => $customersWithOrders,
                'total_orders' => $totalOrders,
                'total_revenue' => $totalRevenue,
                'daily_sales' => $dailySales,
                'weekly_sales' => $weeklySales,
                'monthly_sales' => $monthlySales,
                'yearly_sales' => $yearlySales,
                'low_stock' => $lowStock,
                'out_of_stock' => $outOfStock,
                'total_certificates' => $totalCertificates,
                'today_certificates' => $todayCertificates,
                'active_certificates' => $activeCertificates,
                'revoked_certificates' => $revokedCertificates
            ],
            'top_products' => $topProducts,
            'top_customers' => $topCustomers
        ];
    }
    
    private function getSalesComparison() {
        $period = $_GET['period'] ?? 'month';
        
        $comparison = [];
        
        switch($period) {
            case 'day':
                $current = $this->db->fetchAll("
                    SELECT DATE(created_at) as date, SUM(total_amount) as revenue, COUNT(*) as orders 
                    FROM ".TABLE_PREFIX."orders 
                    WHERE created_at >= CURDATE() AND status = 'completed'
                    GROUP BY DATE(created_at)
                ");
                
                $previous = $this->db->fetchAll("
                    SELECT DATE(created_at) as date, SUM(total_amount) as revenue, COUNT(*) as orders 
                    FROM ".TABLE_PREFIX."orders 
                    WHERE created_at >= DATE_SUB(CURDATE(), INTERVAL 7 DAY) AND created_at < CURDATE() AND status = 'completed'
                    GROUP BY DATE(created_at)
                ");
                break;
                
            case 'week':
                $current = $this->db->fetchAll("
                    SELECT YEAR(created_at) as year, WEEK(created_at) as week, SUM(total_amount) as revenue, COUNT(*) as orders 
                    FROM ".TABLE_PREFIX."orders 
                    WHERE YEAR(created_at) = YEAR(CURDATE()) AND status = 'completed'
                    GROUP BY YEAR(created_at), WEEK(created_at)
                    ORDER BY year, week
                ");
                
                $previous = $this->db->fetchAll("
                    SELECT YEAR(created_at) as year, WEEK(created_at) as week, SUM(total_amount) as revenue, COUNT(*) as orders 
                    FROM ".TABLE_PREFIX."orders 
                    WHERE YEAR(created_at) = YEAR(CURDATE()) - 1 AND status = 'completed'
                    GROUP BY YEAR(created_at), WEEK(created_at)
                    ORDER BY year, week
                ");
                break;
                
            case 'month':
            default:
                $current = $this->db->fetchAll("
                    SELECT YEAR(created_at) as year, MONTH(created_at) as month, SUM(total_amount) as revenue, COUNT(*) as orders 
                    FROM ".TABLE_PREFIX."orders 
                    WHERE YEAR(created_at) = YEAR(CURDATE()) AND status = 'completed'
                    GROUP BY YEAR(created_at), MONTH(created_at)
                ");
                
                $previous = $this->db->fetchAll("
                    SELECT YEAR(created_at) as year, MONTH(created_at) as month, SUM(total_amount) as revenue, COUNT(*) as orders 
                    FROM ".TABLE_PREFIX."orders 
                    WHERE YEAR(created_at) = YEAR(CURDATE()) - 1 AND status = 'completed'
                    GROUP BY YEAR(created_at), MONTH(created_at)
                ");
                break;
        }
        
        return [
            'success' => true,
            'period' => $period,
            'current' => $current,
            'previous' => $previous
        ];
    }
    
    private function getCategories() {
        $categories = $this->db->fetchAll("
            SELECT category, COUNT(*) as product_count 
            FROM ".TABLE_PREFIX."products 
            WHERE category IS NOT NULL AND category != ''
            GROUP BY category 
            ORDER BY product_count DESC
        ");
        
        return ['success' => true, 'categories' => $categories];
    }
    
    private function getAttributes() {
        $products = $this->db->fetchAll("SELECT attributes FROM ".TABLE_PREFIX."products WHERE attributes IS NOT NULL AND attributes != ''");
        
        $attributes = [];
        foreach($products as $product) {
            $productAttrs = json_decode($product['attributes'], true) ?: [];
            foreach($productAttrs as $attr) {
                $attrName = $attr['name'];
                if(!isset($attributes[$attrName])) {
                    $attributes[$attrName] = [];
                }
                if(!empty($attr['options'])) {
                    foreach($attr['options'] as $option) {
                        if(!in_array($option, $attributes[$attrName])) {
                            $attributes[$attrName][] = $option;
                        }
                    }
                }
            }
        }
        
        return ['success' => true, 'attributes' => $attributes];
    }
    
    private function getLogs() {
        $page = intval($_GET['page'] ?? 1);
        $per_page = intval($_GET['per_page'] ?? 50);
        $offset = ($page - 1) * $per_page;
        
        $logs = $this->db->fetchAll(
            "SELECT * FROM ".TABLE_PREFIX."logs ORDER BY created_at DESC LIMIT $offset, $per_page"
        );
        
        $total = $this->db->fetch("SELECT COUNT(*) as total FROM ".TABLE_PREFIX."logs")['total'];
        
        return [
            'success' => true,
            'logs' => $logs,
            'pagination' => [
                'page' => $page,
                'per_page' => $per_page,
                'total' => $total,
                'total_pages' => ceil($total / $per_page)
            ]
        ];
    }
    
    private function getProductLogs() {
        $page = intval($_GET['page'] ?? 1);
        $per_page = intval($_GET['per_page'] ?? 50);
        $product_id = intval($_GET['product_id'] ?? 0);
        $offset = ($page - 1) * $per_page;
        
        $where = $product_id ? "WHERE product_id = $product_id" : "";
        
        $logs = $this->db->fetchAll(
            "SELECT * FROM ".TABLE_PREFIX."product_logs $where ORDER BY created_at DESC LIMIT $offset, $per_page"
        );
        
        $total = $this->db->fetch("SELECT COUNT(*) as total FROM ".TABLE_PREFIX."product_logs $where")['total'];
        
        foreach($logs as &$log) {
            $log['old_data'] = json_decode($log['old_data'], true) ?: [];
            $log['new_data'] = json_decode($log['new_data'], true) ?: [];
            $log['changed_fields'] = json_decode($log['changed_fields'], true) ?: [];
        }
        
        return [
            'success' => true,
            'logs' => $logs,
            'pagination' => [
                'page' => $page,
                'per_page' => $per_page,
                'total' => $total,
                'total_pages' => ceil($total / $per_page)
            ]
        ];
    }
    
    private function saveSettings() {
        $key = $_POST['key'] ?? '';
        $value = $_POST['value'] ?? '';
        
        if(!$key) {
            return ['success' => false, 'message' => 'کلید تنظیمات الزامی است'];
        }
        
        $existing = $this->db->fetch(
            "SELECT id FROM ".TABLE_PREFIX."settings WHERE setting_key = ?", 
            [$key]
        );
        
        if($existing) {
            $this->db->update(TABLE_PREFIX.'settings', ['setting_value' => $value], "setting_key = '$key'");
        } else {
            $this->db->insert(TABLE_PREFIX.'settings', [
                'setting_key' => $key,
                'setting_value' => $value
            ]);
        }
        
        return ['success' => true, 'message' => 'تنظیمات ذخیره شد'];
    }
    
    private function deleteSettings() {
        $key = $_POST['key'] ?? '';
        
        if(!$key) {
            return ['success' => false, 'message' => 'کلید تنظیمات الزامی است'];
        }
        
        $deleted = $this->db->delete(TABLE_PREFIX.'settings', "setting_key = ?", [$key]);
        
        return ['success' => $deleted > 0, 'message' => $deleted > 0 ? 'تنظیمات حذف شد' : 'خطا در حذف تنظیمات'];
    }
    
    private function testConnection() {
        return $this->woo->testConnection();
    }
    
    private function syncProducts() {
        $page = intval($_POST['page'] ?? 1);
        try {
            $result = $this->woo->syncProducts($page);
            return ['success' => true, 'result' => $result];
        } catch(Exception $e) {
            return ['success' => false, 'message' => $e->getMessage()];
        }
    }
    
    private function syncOrders() {
        $page = intval($_POST['page'] ?? 1);
        try {
            $result = $this->woo->syncOrders($page);
            return ['success' => true, 'result' => $result];
        } catch(Exception $e) {
            return ['success' => false, 'message' => $e->getMessage()];
        }
    }

    private function fixDuplicateSKUs() {
        try {
            $result = $this->db->fixDuplicateSKUs();
            return $result;
        } catch(Exception $e) {
            return ['success' => false, 'message' => $e->getMessage()];
        }
    }

    // ** جدید: جستجوی محصولات برای شناسنامه **
    private function searchProductsForCertificate() {
        $query = $_GET['q'] ?? '';
        if(strlen($query) < 2) {
            return ['success' => true, 'products' => []];
        }

        $products = $this->db->fetchAll(
            "SELECT id, sku, name, price, weight, images, attributes FROM ".TABLE_PREFIX."products 
             WHERE sku LIKE ? OR name LIKE ? LIMIT 10",
            ["%$query%", "%$query%"]
        );

        foreach($products as &$p) {
            $p['images'] = json_decode($p['images'], true) ?: [];
            $p['attributes'] = json_decode($p['attributes'], true) ?: [];
        }

        return ['success' => true, 'products' => $products];
    }

    // ** جدید: دریافت ویژگی‌های محصول **
    private function getProductAttributes() {
        $productId = intval($_GET['product_id'] ?? 0);
        $product = $this->db->fetch("SELECT attributes FROM ".TABLE_PREFIX."products WHERE id = ?", [$productId]);
        if(!$product) {
            return ['success' => false, 'message' => 'محصول یافت نشد'];
        }
        $attributes = json_decode($product['attributes'], true) ?: [];
        return ['success' => true, 'attributes' => $attributes];
    }

    // ** جدید: صدور شناسنامه **
    private function issueCertificate() {
        $productId = intval($_POST['product_id'] ?? 0);
        $productSku = $_POST['product_sku'] ?? '';
        $productName = $_POST['product_name'] ?? '';
        $attributes = $_POST['attributes'] ?? [];
        $imageUrl = $_POST['image_url'] ?? '';
        $issueDate = $_POST['issue_date'] ?? date('Y-m-d');

        if($productId == 0 && !empty($productSku) && !empty($productName)) {
            $productData = [
                'sku' => $productSku,
                'name' => $productName,
                'attributes' => json_encode($attributes, JSON_UNESCAPED_UNICODE)
            ];
        } else {
            $product = $this->db->fetch("SELECT * FROM ".TABLE_PREFIX."products WHERE id = ?", [$productId]);
            if(!$product) {
                return ['success' => false, 'message' => 'محصول یافت نشد'];
            }
            $productData = [
                'id' => $product['id'],
                'sku' => $product['sku'],
                'name' => $product['name'],
                'price' => $product['price'],
                'weight' => $product['weight'],
                'dimensions' => [
                    'length' => $product['length'],
                    'width' => $product['width'],
                    'height' => $product['height']
                ],
                'attributes' => json_decode($product['attributes'], true) ?: [],
                'images' => json_decode($product['images'], true) ?: []
            ];
            if(empty($imageUrl) && !empty($productData['images'])) {
                $imageUrl = $productData['images'][0];
            }
            $productName = $productData['name'];
            $productSku = $productData['sku'];
        }

        $certNumber = $this->generateCertificateNumber();
        $shortCode = substr(md5($certNumber . time()), 0, 8);

        $certId = $this->db->saveCertificate([
            'product_id' => $productId,
            'product_sku' => $productSku,
            'product_name' => $productName,
            'product_data' => json_encode($productData, JSON_UNESCAPED_UNICODE),
            'certificate_number' => $certNumber,
            'short_code' => $shortCode,
            'issue_date' => $issueDate,
            'attributes' => json_encode($attributes, JSON_UNESCAPED_UNICODE),
            'image_url' => $imageUrl,
            'qr_url' => 'https://mashahir.jewelry/verify?code=' . $shortCode
        ]);

        $this->db->insert(TABLE_PREFIX.'logs', [
            'action' => 'issue_certificate',
            'description' => "شناسنامه با شماره $certNumber برای محصول $productName صادر شد",
            'user_name' => 'admin'
        ]);

        return [
            'success' => true,
            'certificate' => [
                'id' => $certId,
                'number' => $certNumber,
                'short_code' => $shortCode,
                'issue_date' => $issueDate
            ]
        ];
    }

    private function generateCertificateNumber() {
        do {
            $number = 'CERT-' . date('Ymd') . '-' . rand(1000, 9999);
            $exists = $this->db->fetch("SELECT id FROM ".TABLE_PREFIX."certificates WHERE certificate_number = ?", [$number]);
        } while($exists);
        return $number;
    }

    private function getCertificates() {
        $page = intval($_GET['page'] ?? 1);
        $per_page = intval($_GET['per_page'] ?? 25);
        $filters = [
            'search' => $_GET['search'] ?? '',
            'status' => $_GET['status'] ?? '',
            'date_from' => $_GET['date_from'] ?? '',
            'date_to' => $_GET['date_to'] ?? ''
        ];

        return $this->db->getCertificates($filters, $page, $per_page);
    }

    private function getCertificate() {
        $id = intval($_GET['id'] ?? 0);
        $code = $_GET['code'] ?? '';

        if($id) {
            $cert = $this->db->fetch("SELECT * FROM ".TABLE_PREFIX."certificates WHERE id = ?", [$id]);
        } elseif($code) {
            $cert = $this->db->fetch("SELECT * FROM ".TABLE_PREFIX."certificates WHERE certificate_number = ? OR short_code = ?", [$code, $code]);
        } else {
            return ['success' => false, 'message' => 'شناسه یا کد شناسنامه الزامی است'];
        }

        if(!$cert) {
            return ['success' => false, 'message' => 'شناسنامه یافت نشد'];
        }

        $cert['product_data'] = json_decode($cert['product_data'], true) ?: [];
        $cert['attributes'] = json_decode($cert['attributes'], true) ?: [];

        return ['success' => true, 'certificate' => $cert];
    }

    private function revokeCertificate() {
        $id = intval($_POST['id'] ?? 0);
        if(!$id) {
            return ['success' => false, 'message' => 'شناسه شناسنامه الزامی است'];
        }

        $updated = $this->db->update(TABLE_PREFIX.'certificates', ['status' => 'revoked'], "id = $id");
        if($updated) {
            $this->db->insert(TABLE_PREFIX.'logs', [
                'action' => 'revoke_certificate',
                'description' => "شناسنامه با شناسه $id باطل شد",
                'user_name' => 'admin'
            ]);
            return ['success' => true, 'message' => 'شناسنامه با موفقیت باطل شد'];
        } else {
            return ['success' => false, 'message' => 'خطا در ابطال شناسنامه'];
        }
    }

    private function verifyCertificate() {
        $code = $_GET['code'] ?? '';
        if(!$code) {
            return ['success' => false, 'message' => 'کد شناسنامه الزامی است'];
        }

        $cert = $this->db->fetch(
            "SELECT * FROM ".TABLE_PREFIX."certificates WHERE certificate_number = ? OR short_code = ?",
            [$code, $code]
        );

        if(!$cert) {
            return ['success' => false, 'message' => 'شناسنامه معتبر یافت نشد'];
        }

        if($cert['status'] != 'active') {
            return ['success' => false, 'message' => 'این شناسنامه ' . ($cert['status'] == 'revoked' ? 'باطل شده' : 'منقضی') . ' است'];
        }

        $cert['product_data'] = json_decode($cert['product_data'], true) ?: [];
        $cert['attributes'] = json_decode($cert['attributes'], true) ?: [];

        return ['success' => true, 'certificate' => $cert];
    }
}

// تابع تبدیل تاریخ میلادی به شمسی
function gregorian_to_jalali($gy, $gm, $gd) {
    $g_d_m = array(0,31,59,90,120,151,181,212,243,273,304,334);
    $gy2 = ($gm > 2) ? ($gy + 1) : $gy;
    $days = 355666 + (365 * $gy) + ((int)(($gy2 + 3) / 4)) - ((int)(($gy2 + 99) / 100)) + ((int)(($gy2 + 399) / 400)) + $gd + $g_d_m[$gm-1];
    $jy = -1595 + (33 * ((int)($days / 12053)));
    $days %= 12053;
    $jy += 4 * ((int)($days / 1461));
    $days %= 1461;
    if ($days > 365) {
        $jy += (int)(($days-1) / 365);
        $days = ($days-1) % 365;
    }
    if ($days < 186) {
        $jm = 1 + (int)($days / 31);
        $jd = 1 + ($days % 31);
    } else {
        $jm = 7 + (int)(($days - 186) / 30);
        $jd = 1 + (($days - 186) % 30);
    }
    return array($jy, $jm, $jd);
}

function getCurrentJalaliDate() {
    $date = getdate();
    $jalali = gregorian_to_jalali($date['year'], $date['mon'], $date['mday']);
    
    $weekdays = ['یک‌شنبه', 'دوشنبه', 'سه‌شنبه', 'چهارشنبه', 'پنج‌شنبه', 'جمعه', 'شنبه'];
    $weekday = $weekdays[$date['wday']];
    
    $months = ['', 'فروردین', 'اردیبهشت', 'خرداد', 'تیر', 'مرداد', 'شهریور', 'مهر', 'آبان', 'آذر', 'دی', 'بهمن', 'اسفند'];
    $month_name = $months[$jalali[1]];
    
    return "$weekday " . $jalali[2] . " $month_name " . $jalali[0];
}

function getCurrentJalaliTime() {
    return date('H:i'); // بدون ثانیه
}

// مقداردهی اولیه
$db = new Database();
$woo = new WooCommerceManager($db);
$api = new ShopGunAPI($db, $woo);

// پردازش درخواست‌های API
if(isset($_GET['api']) || isset($_POST['api'])) {
    header('Content-Type: application/json');
    echo json_encode($api->handleRequest());
    exit;
}

// تعیین صفحه فعلی
$current_page = isset($_GET['page']) ? $_GET['page'] : 'dashboard';
?>
<!DOCTYPE html>
<html lang="fa" dir="rtl">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>شاپگان - سیستم مدیریت فروشگاه جواهری</title>
    <link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/@fontsource/vazir@5.0.0/index.min.css">
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.4.0/css/all.min.css">
    <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
    <script src="https://code.jquery.com/jquery-3.6.0.min.js"></script>
    <style>
        /* استایل‌های به‌روز شده با رنگ‌های فیروزه‌ای و قرمز مخملی */
        :root {
            --bg-primary: #1a1a1a;
            --bg-secondary: #2d2d2d;
            --bg-glass: rgba(255, 255, 255, 0.1);
            --text-primary: #e0e0e0;
            --text-secondary: #a0a0a0;
            --accent-turquoise: #40E0D0; /* فیروزه‌ای */
            --accent-crimson: #DC143C;   /* قرمز مخملی */
            --success: #4CAF50;
            --warning: #FF9800;
            --danger: #F44336;
            --border-radius: 12px;
            --transition: all 0.3s ease;
            --shadow: 0 8px 32px rgba(0, 0, 0, 0.3);
            --blur: blur(20px);
        }

        * {
            margin: 0;
            padding: 0;
            box-sizing: border-box;
            font-family: 'Vazir', sans-serif;
        }

        body {
            background: linear-gradient(135deg, var(--bg-primary) 0%, #0d0d0d 100%);
            color: var(--text-primary);
            min-height: 100vh;
            overflow-x: hidden;
        }

        /* تم روشن */
        body.light-theme {
            --bg-primary: #f0f0f0;
            --bg-secondary: #ffffff;
            --bg-glass: rgba(0, 0, 0, 0.05);
            --text-primary: #333333;
            --text-secondary: #666666;
            --shadow: 0 8px 32px rgba(0, 0, 0, 0.1);
        }

        .loading-screen {
            position: fixed;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: var(--bg-primary);
            display: flex;
            flex-direction: column;
            justify-content: center;
            align-items: center;
            z-index: 9999;
            transition: opacity 0.5s ease;
        }

        .loading-logo {
            font-size: 4rem;
            font-weight: bold;
            background: linear-gradient(45deg, var(--accent-turquoise), var(--accent-crimson), var(--accent-turquoise));
            background-size: 200% auto;
            -webkit-background-clip: text;
            -webkit-text-fill-color: transparent;
            background-clip: text;
            animation: shimmer 2s linear infinite;
            margin-bottom: 1rem;
            text-align: center;
        }

        .loading-subtitle {
            font-size: 1.5rem;
            color: var(--text-secondary);
            margin-bottom: 2rem;
            text-align: center;
        }

        .loading-welcome {
            font-size: 1.8rem;
            color: var(--accent-crimson);
            text-align: center;
            animation: bounce 2s infinite;
            margin-bottom: 1rem;
        }

        .loading-version {
            font-size: 1rem;
            color: var(--text-secondary);
            text-align: center;
        }

        @keyframes shimmer {
            0% { background-position: -200% center; }
            100% { background-position: 200% center; }
        }

        @keyframes bounce {
            0%, 100% { transform: translateY(0); }
            50% { transform: translateY(-10px); }
        }

        .container {
            max-width: 1400px;
            margin: 0 auto;
            padding: 20px;
            opacity: 0;
            transform: translateY(20px);
            transition: var(--transition);
        }

        .container.loaded {
            opacity: 1;
            transform: translateY(0);
        }

        .glass-card {
            background: var(--bg-glass);
            backdrop-filter: var(--blur);
            border: 1px solid rgba(255, 255, 255, 0.1);
            border-radius: var(--border-radius);
            padding: 24px;
            margin-bottom: 24px;
            transition: var(--transition);
            box-shadow: var(--shadow);
        }

        .glass-card:hover {
            transform: translateY(-5px);
            box-shadow: 0 15px 40px rgba(0, 0, 0, 0.4);
        }

        .header {
            display: flex;
            justify-content: space-between;
            align-items: center;
            margin-bottom: 30px;
            padding-bottom: 20px;
            border-bottom: 1px solid rgba(255, 255, 255, 0.1);
        }

        .logo-container {
            display: flex;
            align-items: center;
            gap: 15px;
        }

        .logo {
            font-size: 2.5rem;
            font-weight: bold;
            background: linear-gradient(45deg, var(--accent-turquoise), var(--accent-crimson));
            -webkit-background-clip: text;
            -webkit-text-fill-color: transparent;
            transition: var(--transition);
            cursor: pointer;
        }

        .logo:hover {
            transform: scale(1.05);
            text-shadow: 0 0 20px rgba(64, 224, 208, 0.5);
        }

        .logo-info {
            display: flex;
            flex-direction: column;
        }

        .logo-title {
            font-size: 1.2rem;
            font-weight: bold;
            background: linear-gradient(45deg, var(--accent-turquoise), var(--accent-crimson));
            -webkit-background-clip: text;
            -webkit-text-fill-color: transparent;
        }

        .logo-version {
            font-size: 0.9rem;
            color: var(--text-secondary);
        }

        .date-time {
            text-align: left;
            background: var(--bg-glass);
            padding: 15px;
            border-radius: var(--border-radius);
            backdrop-filter: var(--blur);
        }

        .date {
            font-size: 1.2rem;
            font-weight: bold;
            color: var(--accent-turquoise);
        }

        .time {
            font-size: 1rem;
            color: var(--text-secondary);
        }

        .nav-tabs {
            display: flex;
            gap: 10px;
            margin-bottom: 30px;
            flex-wrap: wrap;
        }

        .nav-tab {
            padding: 12px 24px;
            background: var(--bg-secondary);
            border-radius: var(--border-radius);
            cursor: pointer;
            transition: var(--transition);
            border: none;
            color: var(--text-primary);
            font-size: 1rem;
            backdrop-filter: var(--blur);
            text-decoration: none;
            display: inline-block;
            box-shadow: 0 2px 8px rgba(0,0,0,0.1);
        }

        .nav-tab.active {
            background: linear-gradient(135deg, var(--accent-turquoise), var(--accent-crimson));
            color: white;
            box-shadow: 0 4px 15px rgba(64, 224, 208, 0.3);
        }

        .nav-tab:hover:not(.active) {
            background: rgba(64, 224, 208, 0.2);
            transform: translateY(-2px);
        }

        .stats-grid {
            display: grid;
            grid-template-columns: repeat(auto-fit, minmax(280px, 1fr));
            gap: 20px;
            margin-bottom: 30px;
        }

        .stat-card {
            background: var(--bg-glass);
            border-radius: var(--border-radius);
            padding: 25px;
            text-align: center;
            border-left: 4px solid var(--accent-turquoise);
            backdrop-filter: var(--blur);
            transition: var(--transition);
        }

        .stat-card:hover {
            transform: translateY(-5px);
            box-shadow: var(--shadow);
        }

        .stat-value {
            font-size: 2.5rem;
            font-weight: bold;
            color: var(--accent-crimson);
            margin: 10px 0;
        }

        .stat-label {
            color: var(--text-secondary);
            font-size: 0.9rem;
        }

        .comparison-table {
            width: 100%;
            border-collapse: collapse;
            margin: 20px 0;
        }

        .comparison-table th,
        .comparison-table td {
            padding: 15px;
            text-align: center;
            border-bottom: 1px solid rgba(255, 255, 255, 0.1);
        }

        .comparison-table th {
            background: rgba(64, 224, 208, 0.1);
            color: var(--accent-turquoise);
            font-weight: bold;
        }

        .growth-positive {
            color: var(--success);
            font-weight: bold;
        }

        .growth-negative {
            color: var(--danger);
            font-weight: bold;
        }

        .charts-grid {
            display: grid;
            grid-template-columns: 2fr 1fr;
            gap: 20px;
            margin-bottom: 30px;
        }

        @media (max-width: 1024px) {
            .charts-grid {
                grid-template-columns: 1fr;
            }
        }

        .chart-container {
            background: var(--bg-glass);
            border-radius: var(--border-radius);
            padding: 20px;
            height: 400px;
            backdrop-filter: var(--blur);
        }

        /* استایل‌های جدول با سایز خودکار ستون‌ها */
        .table-responsive {
            overflow-x: auto;
            -webkit-overflow-scrolling: touch;
        }

        .table-container {
            background: var(--bg-glass);
            border-radius: var(--border-radius);
            overflow: hidden;
            backdrop-filter: var(--blur);
        }

        table {
            width: 100%;
            border-collapse: collapse;
        }

        th, td {
            padding: 12px 15px;
            text-align: right;
            border-bottom: 1px solid rgba(255, 255, 255, 0.1);
            white-space: nowrap;
        }

        th {
            background: rgba(64, 224, 208, 0.1);
            font-weight: bold;
            color: var(--accent-turquoise);
            cursor: pointer;
            user-select: none;
            position: relative;
        }

        th:hover {
            background: rgba(64, 224, 208, 0.2);
        }

        th.sort-asc::after {
            content: " ↑";
            color: var(--accent-crimson);
        }

        th.sort-desc::after {
            content: " ↓";
            color: var(--accent-crimson);
        }

        tr:hover {
            background: rgba(255, 255, 255, 0.05);
        }

        /* استایل‌های فرم */
        .btn {
            padding: 10px 20px;
            border: none;
            border-radius: var(--border-radius);
            cursor: pointer;
            transition: var(--transition);
            font-size: 0.9rem;
            display: inline-flex;
            align-items: center;
            gap: 8px;
            backdrop-filter: var(--blur);
            text-decoration: none;
        }

        .btn-primary {
            background: linear-gradient(135deg, var(--accent-turquoise), var(--accent-crimson));
            color: white;
            box-shadow: 0 4px 15px rgba(64, 224, 208, 0.3);
        }

        .btn-primary:hover {
            transform: translateY(-2px);
            box-shadow: 0 6px 20px rgba(220, 20, 60, 0.4);
        }

        .btn-secondary {
            background: var(--bg-secondary);
            color: var(--text-primary);
        }

        .btn-secondary:hover {
            background: rgba(255, 255, 255, 0.1);
            transform: translateY(-2px);
        }

        .btn-sm {
            padding: 6px 12px;
            font-size: 0.8rem;
        }

        .form-group {
            margin-bottom: 20px;
            position: relative;
        }

        .form-label {
            display: block;
            margin-bottom: 8px;
            color: var(--text-secondary);
            font-weight: bold;
        }

        .form-control {
            width: 100%;
            padding: 12px;
            background: var(--bg-secondary);
            border: 1px solid rgba(255, 255, 255, 0.1);
            border-radius: var(--border-radius);
            color: var(--text-primary);
            font-size: 1rem;
            transition: var(--transition);
            backdrop-filter: var(--blur);
        }

        .form-control:focus {
            outline: none;
            border-color: var(--accent-turquoise);
            box-shadow: 0 0 0 2px rgba(64, 224, 208, 0.2);
        }

        .search-box {
            position: relative;
            margin-bottom: 20px;
        }

        .search-input {
            padding-right: 40px;
        }

        .search-icon {
            position: absolute;
            right: 12px;
            top: 50%;
            transform: translateY(-50%);
            color: var(--text-secondary);
        }

        .search-results {
            position: absolute;
            background: var(--bg-secondary);
            width: 100%;
            max-height: 200px;
            overflow-y: auto;
            border-radius: var(--border-radius);
            margin-top: 5px;
            z-index: 1000;
            box-shadow: var(--shadow);
        }

        .search-result-item {
            padding: 10px;
            border-bottom: 1px solid rgba(255, 255, 255, 0.1);
            cursor: pointer;
        }

        .search-result-item:hover {
            background: rgba(64, 224, 208, 0.2);
        }

        .pagination {
            display: flex;
            justify-content: center;
            gap: 10px;
            margin-top: 20px;
            flex-wrap: wrap;
        }

        .page-item {
            padding: 8px 16px;
            background: var(--bg-secondary);
            border-radius: var(--border-radius);
            cursor: pointer;
            transition: var(--transition);
            backdrop-filter: var(--blur);
        }

        .page-item.active {
            background: linear-gradient(135deg, var(--accent-turquoise), var(--accent-crimson));
            color: white;
        }

        .page-item:hover:not(.active) {
            background: rgba(64, 224, 208, 0.2);
            transform: translateY(-2px);
        }

        /* استایل‌های badge */
        .badge {
            padding: 4px 12px;
            border-radius: 20px;
            font-size: 0.8rem;
            font-weight: bold;
            display: inline-block;
        }

        .badge-success {
            background: var(--success);
            color: white;
        }

        .badge-warning {
            background: var(--warning);
            color: black;
        }

        .badge-danger {
            background: var(--danger);
            color: white;
        }

        .badge-secondary {
            background: var(--bg-secondary);
            color: var(--text-primary);
        }

        .badge-info {
            background: var(--accent-turquoise);
            color: white;
        }

        .badge-crimson {
            background: var(--accent-crimson);
            color: white;
        }

        /* استایل‌های alert */
        .alert {
            padding: 15px;
            border-radius: var(--border-radius);
            margin-bottom: 20px;
            border-left: 4px solid;
            backdrop-filter: var(--blur);
        }

        .alert-success {
            background: rgba(76, 175, 80, 0.2);
            border-left-color: var(--success);
            color: #c8e6c9;
        }

        .alert-error {
            background: rgba(244, 67, 54, 0.2);
            border-left-color: var(--danger);
            color: #ffcdd2;
        }

        /* استایل‌های مودال */
        .modal {
            display: none;
            position: fixed;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: rgba(0, 0, 0, 0.8);
            z-index: 1000;
            backdrop-filter: blur(10px);
            align-items: center;
            justify-content: center;
        }

        .modal-content {
            background: var(--bg-primary);
            border-radius: var(--border-radius);
            padding: 30px;
            max-width: 90%;
            max-height: 90vh;
            overflow-y: auto;
            box-shadow: var(--shadow);
            border: 1px solid rgba(255, 255, 255, 0.1);
        }

        .close-modal {
            position: absolute;
            top: 15px;
            left: 15px;
            background: none;
            border: none;
            color: var(--text-secondary);
            font-size: 1.5rem;
            cursor: pointer;
            z-index: 1001;
        }

        /* استایل‌های محصولات */
        .product-grid {
            display: grid;
            grid-template-columns: repeat(auto-fill, minmax(300px, 1fr));
            gap: 20px;
        }

        .product-card {
            background: var(--bg-glass);
            border-radius: var(--border-radius);
            padding: 20px;
            transition: var(--transition);
            backdrop-filter: var(--blur);
        }

        .product-card:hover {
            transform: translateY(-5px);
            box-shadow: var(--shadow);
        }

        .product-image {
            width: 100%;
            height: 200px;
            object-fit: cover;
            border-radius: var(--border-radius);
            margin-bottom: 15px;
        }

        .quick-edit-form {
            display: flex;
            gap: 10px;
            align-items: center;
        }

        .quick-edit-input {
            flex: 1;
            padding: 8px;
            background: var(--bg-secondary);
            border: 1px solid rgba(255, 255, 255, 0.1);
            border-radius: var(--border-radius);
            color: var(--text-primary);
        }

        /* استایل‌های سفارشات */
        .order-items {
            margin: 15px 0;
        }

        .order-item {
            display: flex;
            justify-content: space-between;
            align-items: center;
            padding: 10px;
            background: var(--bg-secondary);
            border-radius: var(--border-radius);
            margin-bottom: 10px;
            backdrop-filter: var(--blur);
        }

        /* استایل‌های وضعیت سفارش */
        .status-completed {
            background: rgba(76, 175, 80, 0.2) !important;
            border-left: 4px solid var(--success);
        }

        .status-cancelled {
            background: rgba(244, 67, 54, 0.2) !important;
            border-left: 4px solid var(--danger);
        }

        .status-processing {
            background: rgba(33, 150, 243, 0.2) !important;
            border-left: 4px solid #2196F3;
        }

        .status-pending {
            background: rgba(255, 152, 0, 0.2) !important;
            border-left: 4px solid var(--warning);
        }

        .status-stranger {
            background: rgba(156, 39, 176, 0.2) !important;
            border-left: 4px solid #9C27B0;
        }

        /* استایل‌های برچسب پستی */
        .label-preview {
            width: 120mm;
            height: 60mm;
            padding: 3mm;
            background: #fff;
            color: #000;
            box-sizing: border-box;
            border: 1px solid #000;
            font-family: 'Vazir', sans-serif;
        }

        .label-header {
            display: flex;
            align-items: center;
            justify-content: space-between;
            margin-bottom: 2px;
        }

        .label-name {
            font-weight: 800;
            font-size: 19px;
        }

        .label-insurance {
            font-size: 16px;
            color: #fff;
            background: #000;
            padding: 5px 16px;
        }

        .label-address {
            border: 2px solid #000;
            padding: 3px;
            min-height: 56px;
            white-space: pre-wrap;
            font-size: 19px;
            margin-bottom: 6px;
        }

        .label-contact {
            display: flex;
            justify-content: space-between;
            font-size: 20px;
            margin-bottom: 6px;
        }

        .label-notes {
            font-size: 12px;
            margin-bottom: 6px;
        }

        .label-meta {
            display: flex;
            gap: 8px;
            flex-wrap: wrap;
            font-size: 13px;
            margin-bottom: 8px;
        }

        .label-meta-item {
            border: 1px solid #000;
            padding: 2px;
            border-radius: 4px;
        }

        .label-footer {
            height: 18px;
            background: #000;
            color: #fff;
            text-align: center;
            font-size: 12px;
            padding-top: 1px;
        }

        /* استایل‌های شناسنامه محصول (بر اساس طراحی داده شده) */
        .certificate-card {
            width: 85.6mm;
            height: 54mm;
            position: relative;
            overflow: hidden;
            background: linear-gradient(135deg, #0a0a0a, #1c1c1c);
            color: #fff;
            margin: 0 auto;
            font-family: Arial, sans-serif;
        }

        .security-pattern {
            position: absolute;
            inset: 0;
            background-image: url('logo.png');
            background-size: 18mm;
            background-repeat: repeat;
            opacity: 0.035;
            transform: rotate(-12deg);
            pointer-events: none;
        }

        .watermark {
            position: absolute;
            width: 70mm;
            opacity: 0.06;
            top: 6mm;
            left: 8mm;
        }

        .gold-bar {
            height: 3mm;
            background: linear-gradient(to right, #7a5c1b, #f5d27a, #7a5c1b);
        }

        .certificate-header {
            position: relative;
            padding: 3mm 4mm 1mm 4mm;
        }

        .certificate-title {
            font-size: 13pt;
            font-weight: bold;
            letter-spacing: 1px;
            background: linear-gradient(to bottom, #fff, #d4af37, #8f6b1f);
            -webkit-background-clip: text;
            -webkit-text-fill-color: transparent;
        }

        .certificate-subtitle {
            font-size: 6.5pt;
            color: #ccc;
        }

        .certificate-content {
            display: flex;
            padding: 2mm 4mm;
            gap: 2mm;
        }

        .certificate-info {
            flex: 1;
            font-size: 6.2pt;
        }

        .certificate-row {
            display: flex;
            justify-content: space-between;
            padding: 0.7mm 0;
            border-bottom: 0.2mm solid rgba(212,175,55,0.25);
        }

        .certificate-row:last-child {
            border-bottom: none;
        }

        .certificate-label {
            color: #d4af37;
        }

        .certificate-value {
            font-weight: bold;
        }

        .certificate-image-box {
            width: 22mm;
            height: 22mm;
            border: 0.4mm solid #d4af37;
            border-radius: 2mm;
            overflow: hidden;
        }

        .certificate-footer {
            position: absolute;
            bottom: 2mm;
            left: 4mm;
            right: 4mm;
            display: flex;
            justify-content: space-between;
            align-items: flex-end;
        }

        .certificate-qr img {
            width: 13mm;
        }

        .certificate-secret {
            position: absolute;
            bottom: 1.5mm;
            right: 3mm;
            font-size: 5pt;
            color: rgba(255,255,255,0.35);
        }

        .certificate-microtext {
            position: absolute;
            bottom: 6mm;
            left: 4mm;
            font-size: 4pt;
            letter-spacing: 0.3mm;
            opacity: 0.25;
        }

        /* استایل‌های responsive */
        @media (max-width: 768px) {
            .header {
                flex-direction: column;
                gap: 15px;
                text-align: center;
            }

            .logo-container {
                justify-content: center;
            }

            .date-time {
                text-align: center;
            }

            .nav-tabs {
                justify-content: center;
            }

            .stats-grid {
                grid-template-columns: 1fr;
            }

            .modal-content {
                padding: 20px;
                max-width: 95%;
            }

            .label-preview {
                width: 100%;
                height: auto;
                transform: scale(0.8);
                transform-origin: top center;
            }
        }

        /* استایل‌های utility */
        .fade-in {
            animation: fadeIn 0.5s ease-in;
        }

        @keyframes fadeIn {
            from { opacity: 0; transform: translateY(20px); }
            to { opacity: 1; transform: translateY(0); }
        }

        .settings-grid {
            display: grid;
            grid-template-columns: repeat(auto-fit, minmax(300px, 1fr));
            gap: 20px;
        }

        .setting-card {
            background: var(--bg-glass);
            border-radius: var(--border-radius);
            padding: 20px;
            backdrop-filter: var(--blur);
        }

        .color-picker {
            width: 50px;
            height: 30px;
            border: none;
            border-radius: var(--border-radius);
            cursor: pointer;
            background: var(--bg-secondary);
        }

        .glass-effect {
            background: var(--bg-glass);
            backdrop-filter: var(--blur);
            -webkit-backdrop-filter: var(--blur);
        }

        .text-turquoise {
            color: var(--accent-turquoise);
        }

        .text-crimson {
            color: var(--accent-crimson);
        }

        .text-muted {
            color: var(--text-secondary);
        }

        .text-center {
            text-align: center;
        }

        .mb-3 {
            margin-bottom: 1rem;
        }

        .mt-3 {
            margin-top: 1rem;
        }

        .d-none {
            display: none;
        }

        .d-flex {
            display: flex;
        }

        .d-block {
            display: block;
        }

        .justify-content-between {
            justify-content: space-between;
        }

        .align-items-center {
            align-items: center;
        }

        .gap-3 {
            gap: 1rem;
        }

        .w-100 {
            width: 100%;
        }

        .row {
            display: flex;
            flex-wrap: wrap;
            margin: 0 -10px;
        }

        .col-6 {
            flex: 0 0 50%;
            padding: 0 10px;
        }

        @media (max-width: 768px) {
            .col-6 {
                flex: 0 0 100%;
            }
        }

        /* استایل‌های فیلتر و جستجو */
        .filter-section {
            background: var(--bg-glass);
            border-radius: var(--border-radius);
            padding: 20px;
            margin-bottom: 20px;
            backdrop-filter: var(--blur);
        }

        .filter-row {
            display: flex;
            gap: 15px;
            flex-wrap: wrap;
            align-items: end;
        }

        .filter-group {
            flex: 1;
            min-width: 200px;
        }

        /* استایل‌های پیشرفت همگام‌سازی */
        .sync-progress {
            background: var(--bg-glass);
            border-radius: var(--border-radius);
            padding: 15px;
            margin-bottom: 20px;
            backdrop-filter: var(--blur);
        }

        .progress-bar {
            width: 100%;
            height: 10px;
            background: var(--bg-secondary);
            border-radius: 5px;
            overflow: hidden;
            margin: 10px 0;
        }

        .progress-fill {
            height: 100%;
            background: linear-gradient(90deg, var(--accent-turquoise), var(--accent-crimson));
            width: 0%;
            transition: width 0.3s;
        }

        .sync-stats {
            display: flex;
            justify-content: space-between;
            font-size: 0.9rem;
            color: var(--text-secondary);
        }
    </style>
</head>
<body>
    <!-- صفحه لودینگ (فقط برای صفحه اول) -->
    <div class="loading-screen" id="loadingScreen">
        <div class="loading-logo">شاپگان - ShopGun</div>
        <div class="loading-welcome">امیر😎: به قلمرو من خوش آمدید</div>
        <div class="loading-subtitle">سیستم مدیریت فروشگاه جواهری</div>
        <div class="loading-version">نسخه ۴.۰ زمستان ۱۴۰۴ - با شناسنامه محصول</div>
    </div>

    <!-- محتوای اصلی -->
    <div class="container" id="mainContainer">
        <div class="header">
            <div class="logo-container">
                <div class="logo">💎</div>
                <div class="logo-info">
                    <div class="logo-title">شاپگان</div>
                    <div class="logo-version">نسخه ۴.۰ | ساخت: بهمن ۱۴۰۴ | بروزرسانی: <?php echo getCurrentJalaliDate(); ?></div>
                </div>
            </div>
            <div class="date-time">
                <div class="date" id="currentDate"><?php echo getCurrentJalaliDate(); ?></div>
                <div class="time" id="currentTime"><?php echo getCurrentJalaliTime(); ?></div>
            </div>
        </div>

        <div class="nav-tabs">
            <a href="?page=dashboard" class="nav-tab <?php echo $current_page == 'dashboard' ? 'active' : ''; ?>">داشبورد</a>
            <a href="?page=products" class="nav-tab <?php echo $current_page == 'products' ? 'active' : ''; ?>">محصولات</a>
            <a href="?page=orders" class="nav-tab <?php echo $current_page == 'orders' ? 'active' : ''; ?>">سفارشات</a>
            <a href="?page=new-order" class="nav-tab <?php echo $current_page == 'new-order' ? 'active' : ''; ?>">سفارش جدید</a>
            <a href="?page=stranger-order" class="nav-tab <?php echo $current_page == 'stranger-order' ? 'active' : ''; ?>">سفارش بیگانگان</a>
            <a href="?page=certificates" class="nav-tab <?php echo $current_page == 'certificates' ? 'active' : ''; ?>">مدیریت شناسنامه</a>
            <a href="?page=issue-certificate" class="nav-tab <?php echo $current_page == 'issue-certificate' ? 'active' : ''; ?>">صدور شناسنامه</a>
            <a href="?page=logs" class="nav-tab <?php echo $current_page == 'logs' ? 'active' : ''; ?>">لاگ‌ها</a>
            <a href="?page=settings" class="nav-tab <?php echo $current_page == 'settings' ? 'active' : ''; ?>">تنظیمات</a>
        </div>

        <!-- محتوای صفحات -->
        <div id="pageContent">
            <?php
            switch($current_page) {
                case 'dashboard':
                    ?>
                    <div class="page-content active" id="dashboardPage">
                        <div class="stats-grid">
                            <div class="stat-card">
                                <i class="fas fa-box fa-2x text-turquoise"></i>
                                <div class="stat-value" id="totalProducts">-</div>
                                <div class="stat-label">کل محصولات</div>
                            </div>
                            <div class="stat-card">
                                <i class="fas fa-dollar-sign fa-2x text-turquoise"></i>
                                <div class="stat-value" id="productsValue">-</div>
                                <div class="stat-label">ارزش موجودی</div>
                            </div>
                            <div class="stat-card">
                                <i class="fas fa-users fa-2x text-turquoise"></i>
                                <div class="stat-value" id="totalCustomers">-</div>
                                <div class="stat-label">مشتریان</div>
                            </div>
                            <div class="stat-card">
                                <i class="fas fa-shopping-cart fa-2x text-turquoise"></i>
                                <div class="stat-value" id="totalOrders">-</div>
                                <div class="stat-label">سفارشات</div>
                            </div>
                            <div class="stat-card">
                                <i class="fas fa-chart-line fa-2x text-turquoise"></i>
                                <div class="stat-value" id="totalRevenue">-</div>
                                <div class="stat-label">درآمد کل</div>
                            </div>
                            <div class="stat-card">
                                <i class="fas fa-exclamation-triangle fa-2x text-turquoise"></i>
                                <div class="stat-value" id="stockStatus">-</div>
                                <div class="stat-label" id="stockDetails">-</div>
                            </div>
                            <div class="stat-card">
                                <i class="fas fa-certificate fa-2x text-turquoise"></i>
                                <div class="stat-value" id="totalCertificates">-</div>
                                <div class="stat-label">کل شناسنامه‌ها</div>
                            </div>
                            <div class="stat-card">
                                <i class="fas fa-check-circle fa-2x text-turquoise"></i>
                                <div class="stat-value" id="activeCertificates">-</div>
                                <div class="stat-label">شناسنامه فعال</div>
                            </div>
                        </div>

                        <div class="charts-grid">
                            <div class="chart-container">
                                <canvas id="salesChart"></canvas>
                            </div>
                            <div class="chart-container">
                                <canvas id="topProductsChart"></canvas>
                            </div>
                        </div>

                        <div class="glass-card">
                            <h3 class="text-turquoise mb-3">مقایسه فروش</h3>
                            <div class="table-responsive">
                                <table class="comparison-table">
                                    <thead>
                                        <tr>
                                            <th>دوره</th>
                                            <th>فروش امسال</th>
                                            <th>فروش سال قبل</th>
                                            <th>رشد</th>
                                        </tr>
                                    </thead>
                                    <tbody id="salesComparisonTable"></tbody>
                                </table>
                            </div>
                        </div>

                        <div class="row">
                            <div class="col-6">
                                <div class="glass-card">
                                    <h3 class="text-turquoise mb-3">محصولات پرفروش</h3>
                                    <div class="table-responsive">
                                        <table>
                                            <thead>
                                                <tr>
                                                    <th>نام محصول</th>
                                                    <th>SKU</th>
                                                    <th>تعداد فروش</th>
                                                    <th>موجودی</th>
                                                </tr>
                                            </thead>
                                            <tbody id="topProductsTable"></tbody>
                                        </table>
                                    </div>
                                </div>
                            </div>
                            <div class="col-6">
                                <div class="glass-card">
                                    <h3 class="text-turquoise mb-3">مشتریان برتر</h3>
                                    <div class="table-responsive">
                                        <table>
                                            <thead>
                                                <tr>
                                                    <th>نام</th>
                                                    <th>تلفن</th>
                                                    <th>تعداد سفارش</th>
                                                    <th>مبلغ خرید</th>
                                                </tr>
                                            </thead>
                                            <tbody id="topCustomersTable"></tbody>
                                        </table>
                                    </div>
                                </div>
                            </div>
                        </div>
                    </div>
                    <?php
                    break;
                case 'products':
                    ?>
                    <div class="page-content active" id="productsPage">
                        <div class="filter-section">
                            <div class="filter-row">
                                <div class="filter-group">
                                    <label class="form-label">جستجوی محصول</label>
                                    <div class="search-box">
                                        <i class="fas fa-search search-icon"></i>
                                        <input type="text" class="form-control search-input" id="productSearch" placeholder="نام ...">
                                    </div>
                                </div>
                                <div class="filter-group">
                                    <label class="form-label">دسته‌بندی</label>
                                    <select class="form-control" id="productCategoryFilter">
                                        <option value="">همه دسته‌ها</option>
                                    </select>
                                </div>
                                <div class="filter-group">
                                    <label class="form-label">ویژگی</label>
                                    <select class="form-control" id="productAttributeFilter">
                                        <option value="">همه ویژگی‌ها</option>
                                    </select>
                                </div>
                                <div>
                                    <button class="btn btn-primary" onclick="syncProducts()" id="syncProductsBtn">همگام‌سازی</button>
                                    <button class="btn btn-secondary" onclick="fullSyncProducts()" id="fullSyncProducts">همگام‌سازی کامل</button>
                                </div>
                            </div>
                        </div>

                        <div class="glass-card">
                            <h3 class="text-turquoise mb-3">لیست محصولات</h3>
                            <div class="table-responsive">
                                <table id="productsTable">
                                    <thead>
                                        <tr>
                                            <th data-sort="name">نام محصول</th>
                                            <th data-sort="price">قیمت</th>
                                            <th>قیمت هر گرم</th>
                                            <th data-sort="stock">موجودی</th>
                                            <th data-sort="category">دسته‌بندی</th>
                                            <th>عملیات</th>
                                        </tr>
                                    </thead>
                                    <tbody></tbody>
                                </table>
                            </div>
                            <div class="pagination" id="productsPagination"></div>
                        </div>

                        <div class="glass-card">
                            <h3 class="text-turquoise mb-3">بروزرسانی گروهی قیمت (بر اساس هر گرم)</h3>
                            <div class="row">
                                <div class="col-6">
                                    <div class="form-group">
                                        <label class="form-label">دسته‌بندی</label>
                                        <select class="form-control" id="bulkCategory">
                                            <option value="">همه دسته‌ها</option>
                                        </select>
                                    </div>
                                </div>
                                <div class="col-6">
                                    <div class="form-group">
                                        <label class="form-label">ویژگی</label>
                                        <select class="form-control" id="bulkAttribute">
                                            <option value="">همه ویژگی‌ها</option>
                                        </select>
                                    </div>
                                </div>
                                <div class="col-6">
                                    <div class="form-group">
                                        <label class="form-label">قیمت هر گرم جدید (تومان)</label>
                                        <input type="number" class="form-control" id="bulkPricePerGram" step="100">
                                    </div>
                                </div>
                                <div class="col-6">
                                    <div class="form-group">
                                        <label class="form-label">محدودیت تعداد</label>
                                        <input type="number" class="form-control" id="bulkLimit" value="0">
                                    </div>
                                </div>
                                <div>
                                    <button class="btn btn-primary" id="applyBulkPricePerGram">اعمال قیمت هر گرم</button>
                                </div>
                            </div>
                        </div>
                    </div>
                    <?php
                    break;
                case 'orders':
                    ?>
                    <div class="page-content active" id="ordersPage">
                        <div class="filter-section">
                            <div class="filter-row">
                                <div class="filter-group">
                                    <label class="form-label">جستجوی سفارش</label>
                                    <div class="search-box">
                                        <i class="fas fa-search search-icon"></i>
                                        <input type="text" class="form-control search-input" id="orderSearch" placeholder="شماره سفارش یا نام مشتری ...">
                                    </div>
                                </div>
                                <div class="filter-group">
                                    <label class="form-label">وضعیت</label>
                                    <select class="form-control" id="orderStatusFilter">
                                        <option value="">همه وضعیت‌ها</option>
                                        <option value="pending">در انتظار</option>
                                        <option value="processing">در حال پردازش</option>
                                        <option value="completed">تکمیل شده</option>
                                        <option value="cancelled">لغو شده</option>
                                        <option value="stranger">بیگانگان</option>
                                    </select>
                                </div>
                                <div>
                                    <button class="btn btn-primary" onclick="syncOrders()" id="syncOrdersBtn">همگام‌سازی</button>
                                    <button class="btn btn-secondary" onclick="fullSyncOrders()" id="fullSyncOrders">همگام‌سازی کامل</button>
                                </div>
                            </div>
                        </div>

                        <div class="glass-card">
                            <h3 class="text-turquoise mb-3">لیست سفارشات</h3>
                            <div class="table-responsive">
                                <table>
                                    <thead>
                                        <tr>
                                            <th>شماره سفارش</th>
                                            <th>مشتری</th>
                                            <th>مبلغ کل</th>
                                            <th>وضعیت</th>
                                            <th>تاریخ</th>
                                            <th>عملیات</th>
                                        </tr>
                                    </thead>
                                    <tbody id="ordersTable"></tbody>
                                </table>
                            </div>
                            <div class="pagination" id="ordersPagination"></div>
                        </div>
                    </div>
                    <?php
                    break;
                case 'new-order':
                    ?>
                    <div class="page-content active" id="newOrderPage">
                        <div class="glass-card">
                            <h3 class="text-turquoise mb-3">ایجاد سفارش جدید</h3>
                            <div class="form-group">
                                <label class="form-label">جستجوی مشتری با شماره تلفن</label>
                                <input type="text" class="form-control" id="customerPhoneSearch" placeholder="شماره تلفن را وارد کنید">
                            </div>
                            <div id="customerInfo" class="d-none">
                                <div class="form-group">
                                    <label class="form-label">نام مشتری</label>
                                    <input type="text" class="form-control" id="customerName">
                                </div>
                                <div class="form-group">
                                    <label class="form-label">شماره تلفن</label>
                                    <input type="text" class="form-control" id="customerPhone" readonly>
                                </div>
                                <div class="form-group">
                                    <label class="form-label">آدرس</label>
                                    <input type="text" class="form-control" id="customerAddress">
                                </div>
                            </div>
                            <hr>
                            <div class="form-group">
                                <label class="form-label">جستجوی محصول (SKU)</label>
                                <input type="text" class="form-control" id="productSearchOrder" placeholder="SKU محصول را وارد کنید">
                            </div>
                            <div class="order-items" id="orderItems"></div>
                            <div class="d-flex justify-content-between align-items-center">
                                <h4>جمع کل: <span id="orderTotal">0</span> تومان</h4>
                                <button class="btn btn-primary" id="submitOrder">ثبت سفارش</button>
                            </div>
                        </div>
                    </div>
                    <?php
                    break;
                case 'stranger-order':
                    ?>
                    <div class="page-content active" id="strangerOrderPage">
                        <div class="glass-card">
                            <h3 class="text-turquoise mb-3">سفارش بیگانگان</h3>
                            <div class="form-group">
                                <label class="form-label">جستجوی مشتری با شماره تلفن</label>
                                <input type="text" class="form-control" id="customerPhoneSearch" placeholder="شماره تلفن را وارد کنید">
                                <div id="customerSearchResults" class="search-results" style="display: none;"></div>
                            </div>
                            <div id="strangerCustomerInfo">
                                <div class="row">
                                    <div class="col-6">
                                        <div class="form-group">
                                            <label class="form-label">نام مشتری</label>
                                            <input type="text" class="form-control" id="strangerCustomerName">
                                        </div>
                                    </div>
                                    <div class="col-6">
                                        <div class="form-group">
                                            <label class="form-label">شماره تلفن</label>
                                            <input type="text" class="form-control" id="strangerCustomerPhone">
                                        </div>
                                    </div>
                                </div>
                                <div class="form-group">
                                    <label class="form-label">آدرس</label>
                                    <input type="text" class="form-control" id="strangerCustomerAddress">
                                </div>
                                <div class="row">
                                    <div class="col-6">
                                        <div class="form-group">
                                            <label class="form-label">شهر</label>
                                            <input type="text" class="form-control" id="strangerCustomerCity">
                                        </div>
                                    </div>
                                    <div class="col-6">
                                        <div class="form-group">
                                            <label class="form-label">استان</label>
                                            <input type="text" class="form-control" id="strangerCustomerProvince">
                                        </div>
                                    </div>
                                </div>
                                <div class="row">
                                    <div class="col-6">
                                        <div class="form-group">
                                            <label class="form-label">کد پستی</label>
                                            <input type="text" class="form-control" id="strangerCustomerPostal">
                                        </div>
                                    </div>
                                    <div class="col-6">
                                        <div class="form-group">
                                            <label class="form-label">یادداشت</label>
                                            <input type="text" class="form-control" id="strangerCustomerNotes">
                                        </div>
                                    </div>
                                </div>
                            </div>
                            <hr>
                            <div class="form-group">
                                <label class="form-label">جستجوی محصول (نام یا SKU)</label>
                                <input type="text" class="form-control" id="strangerProductSearch" placeholder="نام یا SKU ...">
                                <div id="productSearchResults" class="search-results" style="display: none;"></div>
                            </div>
                            <div class="order-items" id="strangerOrderItems"></div>
                            <div class="d-flex justify-content-between align-items-center">
                                <h4>جمع کل: <span id="strangerOrderTotal">0</span> تومان</h4>
                                <button class="btn btn-primary" id="submitStrangerOrder">ثبت سفارش بیگانگان</button>
                            </div>
                        </div>
                    </div>
                    <?php
                    break;
                case 'certificates':
                    ?>
                    <div class="page-content active" id="certificatesPage">
                        <div class="stats-grid">
                            <div class="stat-card">
                                <i class="fas fa-certificate fa-2x text-turquoise"></i>
                                <div class="stat-value" id="totalCertificates">-</div>
                                <div class="stat-label">کل شناسنامه‌ها</div>
                            </div>
                            <div class="stat-card">
                                <i class="fas fa-calendar-day fa-2x text-turquoise"></i>
                                <div class="stat-value" id="todayCertificates">-</div>
                                <div class="stat-label">صدور امروز</div>
                            </div>
                            <div class="stat-card">
                                <i class="fas fa-check-circle fa-2x text-turquoise"></i>
                                <div class="stat-value" id="activeCertificates">-</div>
                                <div class="stat-label">فعال</div>
                            </div>
                            <div class="stat-card">
                                <i class="fas fa-ban fa-2x text-turquoise"></i>
                                <div class="stat-value" id="revokedCertificates">-</div>
                                <div class="stat-label">باطل شده</div>
                            </div>
                        </div>

                        <div class="charts-grid">
                            <div class="chart-container">
                                <canvas id="certificatesChart"></canvas>
                            </div>
                            <div class="glass-card">
                                <h3 class="text-turquoise">آخرین فعالیت‌ها</h3>
                                <div id="recentActivity"></div>
                            </div>
                        </div>

                        <div class="filter-section">
                            <div class="filter-row">
                                <div class="filter-group">
                                    <label class="form-label">جستجو</label>
                                    <input type="text" class="form-control" id="certSearch" placeholder="کد شناسنامه، محصول...">
                                </div>
                                <div class="filter-group">
                                    <label class="form-label">وضعیت</label>
                                    <select class="form-control" id="certStatusFilter">
                                        <option value="">همه</option>
                                        <option value="active">فعال</option>
                                        <option value="revoked">باطل شده</option>
                                    </select>
                                </div>
                                <div class="filter-group">
                                    <label class="form-label">از تاریخ</label>
                                    <input type="date" class="form-control" id="certDateFrom">
                                </div>
                                <div class="filter-group">
                                    <label class="form-label">تا تاریخ</label>
                                    <input type="date" class="form-control" id="certDateTo">
                                </div>
                                <div>
                                    <button class="btn btn-primary" onclick="loadCertificates()">اعمال فیلتر</button>
                                </div>
                            </div>
                        </div>

                        <div class="glass-card">
                            <h3 class="text-turquoise mb-3">لیست شناسنامه‌ها</h3>
                            <div class="table-responsive">
                                <table>
                                    <thead>
                                        <tr>
                                            <th>تصویر</th>
                                            <th>عنوان محصول</th>
                                            <th>دسته</th>
                                            <th>تاریخ صدور</th>
                                            <th>کد کوتاه</th>
                                            <th>شماره شناسنامه</th>
                                            <th>وضعیت</th>
                                            <th>عملیات</th>
                                        </tr>
                                    </thead>
                                    <tbody id="certificatesTable"></tbody>
                                </table>
                            </div>
                            <div class="pagination" id="certificatesPagination"></div>
                        </div>
                    </div>
                    <?php
                    break;
                case 'issue-certificate':
                    ?>
                    <div class="page-content active" id="issueCertificatePage">
                        <div class="glass-card">
                            <h3 class="text-turquoise mb-3">صدور شناسنامه جدید</h3>
                            <div class="form-group">
                                <label class="form-label">جستجوی محصول (نام یا SKU)</label>
                                <input type="text" class="form-control" id="certProductSearch" placeholder="نام یا SKU ...">
                                <div id="certProductResults" class="search-results" style="display: none;"></div>
                            </div>

                            <form id="certificateForm">
                                <input type="hidden" id="certProductId" value="0">
                                <div class="row">
                                    <div class="col-6">
                                        <div class="form-group">
                                            <label class="form-label">نام محصول</label>
                                            <input type="text" class="form-control" id="certProductName" required>
                                        </div>
                                    </div>
                                    <div class="col-6">
                                        <div class="form-group">
                                            <label class="form-label">SKU</label>
                                            <input type="text" class="form-control" id="certProductSku" required>
                                        </div>
                                    </div>
                                </div>
                                <div class="row">
                                    <div class="col-6">
                                        <div class="form-group">
                                            <label class="form-label">وزن (گرم)</label>
                                            <input type="number" step="0.01" class="form-control" id="certWeight">
                                        </div>
                                    </div>
                                    <div class="col-6">
                                        <div class="form-group">
                                            <label class="form-label">ابعاد (سانتی‌متر)</label>
                                            <div class="d-flex gap-2">
                                                <input type="number" step="0.1" class="form-control" placeholder="طول" id="certLength">
                                                <input type="number" step="0.1" class="form-control" placeholder="عرض" id="certWidth">
                                                <input type="number" step="0.1" class="form-control" placeholder="ارتفاع" id="certHeight">
                                            </div>
                                        </div>
                                    </div>
                                </div>
                                <div class="form-group">
                                    <label class="form-label">نوع فلز</label>
                                    <input type="text" class="form-control" id="certMetal" placeholder="مثال: طلا 18 عیار">
                                </div>
                                <div class="form-group">
                                    <label class="form-label">اصالت سنگ</label>
                                    <select class="form-control" id="certStoneAuthenticity">
                                        <option value="طبیعی">طبیعی</option>
                                        <option value="ترکیبی">ترکیبی</option>
                                        <option value="مصنوعی">مصنوعی</option>
                                    </select>
                                </div>
                                <div class="form-group">
                                    <label class="form-label">تصویر محصول</label>
                                    <input type="text" class="form-control" id="certImageUrl" placeholder="آدرس تصویر">
                                </div>
                                <div class="form-group">
                                    <label class="form-label">تاریخ صدور (شمسی)</label>
                                    <input type="text" class="form-control" id="certIssueDate" value="<?php echo getCurrentJalaliDate(); ?>" readonly>
                                </div>
                                <div class="d-flex gap-3">
                                    <button type="button" class="btn btn-secondary" onclick="previewCertificate()">پیش‌نمایش</button>
                                    <button type="button" class="btn btn-primary" onclick="issueCertificate()">صدور شناسنامه</button>
                                </div>
                            </form>
                        </div>

                        <div class="glass-card" id="certificatePreviewArea" style="display: none;">
                            <h3 class="text-turquoise mb-3">پیش‌نمایش</h3>
                            <div id="certificatePreview"></div>
                        </div>
                    </div>
                    <?php
                    break;
                case 'logs':
                    ?>
                    <div class="page-content active" id="logsPage">
                        <div class="filter-section">
                            <div class="filter-row">
                                <button class="btn btn-secondary" data-log-type="system">لاگ‌های سیستم</button>
                                <button class="btn btn-secondary" data-log-type="product">لاگ‌های محصولات</button>
                            </div>
                        </div>

                        <div id="systemLogsSection">
                            <div class="glass-card">
                                <h3 class="text-turquoise mb-3">لاگ‌های سیستم</h3>
                                <div class="table-responsive">
                                    <table>
                                        <thead>
                                            <tr>
                                                <th>تاریخ</th>
                                                <th>عملیات</th>
                                                <th>توضیحات</th>
                                                <th>کاربر</th>
                                            </tr>
                                        </thead>
                                        <tbody id="systemLogsTable"></tbody>
                                    </table>
                                </div>
                                <div class="pagination" id="systemLogsPagination"></div>
                            </div>
                        </div>

                        <div id="productLogsSection" class="d-none">
                            <div class="glass-card">
                                <h3 class="text-turquoise mb-3">لاگ‌های محصولات</h3>
                                <div class="table-responsive">
                                    <table>
                                        <thead>
                                            <tr>
                                                <th>تاریخ</th>
                                                <th>محصول</th>
                                                <th>عملیات</th>
                                                <th>فیلدهای تغییر یافته</th>
                                                <th>کاربر</th>
                                            </tr>
                                        </thead>
                                        <tbody id="productLogsTable"></tbody>
                                    </table>
                                </div>
                                <div class="pagination" id="productLogsPagination"></div>
                            </div>
                        </div>
                    </div>
                    <?php
                    break;
                case 'settings':
                    ?>
                    <div class="page-content active" id="settingsPage">
                        <div class="settings-grid">
                            <div class="setting-card">
                                <h3 class="text-turquoise mb-3">تنظیمات WooCommerce</h3>
                                <div class="form-group">
                                    <label class="form-label">آدرس فروشگاه</label>
                                    <input type="text" class="form-control" id="storeUrl" placeholder="https://example.com">
                                </div>
                                <div class="form-group">
                                    <label class="form-label">Consumer Key</label>
                                    <input type="text" class="form-control" id="consumerKey">
                                </div>
                                <div class="form-group">
                                    <label class="form-label">Consumer Secret</label>
                                    <input type="text" class="form-control" id="consumerSecret">
                                </div>
                                <div class="d-flex gap-3">
                                    <button class="btn btn-primary" id="testConnection">تست اتصال</button>
                                    <button class="btn btn-secondary" id="saveWooSettings">ذخیره</button>
                                </div>
                            </div>

                            <div class="setting-card">
                                <h3 class="text-turquoise mb-3">تنظیمات ظاهری</h3>
                                <div class="form-group">
                                    <label class="form-label">رنگ اصلی</label>
                                    <input type="color" class="color-picker" id="primaryColor" value="#40E0D0">
                                </div>
                                <div class="form-group">
                                    <label class="form-label">قالب</label>
                                    <select class="form-control" id="themeSelect">
                                        <option value="dark">تاریک</option>
                                        <option value="light">روشن</option>
                                    </select>
                                </div>
                                <button class="btn btn-primary" id="saveAppearance">ذخیره</button>
                            </div>
                        </div>
                    </div>
                    <?php
                    break;
                default:
                    echo '<div class="page-content active">لطفاً یک صفحه را انتخاب کنید.</div>';
            }
            ?>
        </div>
    </div>

    <!-- مودال‌ها -->
    <div class="modal" id="orderModal">
        <div class="modal-content">
            <button class="close-modal">&times;</button>
            <h3 class="text-turquoise mb-3">جزئیات سفارش</h3>
            <div id="orderModalContent"></div>
        </div>
    </div>

    <div class="modal" id="productModal">
        <div class="modal-content">
            <button class="close-modal">&times;</button>
            <h3 class="text-turquoise mb-3">مشاهده و ویرایش محصول</h3>
            <div id="productModalContent"></div>
        </div>
    </div>

    <div class="modal" id="labelModal">
        <div class="modal-content">
            <button class="close-modal">&times;</button>
            <h3 class="text-turquoise mb-3">پیش‌نمایش برچسب پستی (60×100)</h3>
            <div id="labelPreview" class="label-preview"></div>
            <div class="d-flex justify-content-end mt-3">
                <button class="btn btn-primary" onclick="printLabel()">چاپ برچسب</button>
            </div>
        </div>
    </div>

    <div class="modal" id="certificateModal">
        <div class="modal-content">
            <button class="close-modal">&times;</button>
            <h3 class="text-turquoise mb-3">شناسنامه محصول</h3>
            <div id="certificateModalContent"></div>
            <div class="d-flex justify-content-end mt-3 gap-3">
                <button class="btn btn-secondary" onclick="printCertificate()">چاپ</button>
                <button class="btn btn-primary" onclick="downloadCertificate()">دانلود PDF</button>
            </div>
        </div>
    </div>

    <script>
        // متغیرهای سراسری
        const appState = {
            currentPage: '<?php echo $current_page; ?>',
            currentProductPage: 1,
            currentOrderPage: 1,
            currentCertPage: 1,
            currentSystemLogPage: 1,
            currentProductLogPage: 1,
            strangerOrderItems: [],
            productSort: { field: 'id', order: 'DESC' }
        };

        $(document).ready(function() {
            updateDateTime();
            setInterval(updateDateTime, 1000);

            setTimeout(() => {
                $('#loadingScreen').fadeOut(1000);
                $('#mainContainer').addClass('loaded');
                loadCurrentPage();
            }, 2000);

            $(window).click(function(event) {
                if ($(event.target).hasClass('modal')) {
                    $(event.target).hide();
                }
            });

            $('.close-modal').click(function() {
                $(this).closest('.modal').hide();
            });

            // صفحه سفارش بیگانگان
            $('#customerPhoneSearch').on('input', debounce(searchCustomer, 500));
            $('#strangerProductSearch').on('input', debounce(searchProductForStranger, 500));
            $('#submitStrangerOrder').click(createStrangerOrder);

            // صفحه محصولات
            $('#productSearch').on('input', debounce(() => loadProducts(1), 500));
            $('#productCategoryFilter').change(() => loadProducts(1));
            $('#productAttributeFilter').change(() => loadProducts(1));
            $('#productsTable thead th[data-sort]').click(function() {
                const field = $(this).data('sort');
                const currentOrder = appState.productSort.field === field ? (appState.productSort.order === 'ASC' ? 'DESC' : 'ASC') : 'ASC';
                appState.productSort = { field, order: currentOrder };
                loadProducts(1);
            });
            $('#applyBulkPricePerGram').click(applyBulkPricePerGram);
            $('#syncProductsBtn').click(syncProducts);
            $('#fullSyncProducts').click(fullSyncProducts);

            // صفحه سفارشات
            $('#orderSearch').on('input', debounce(() => loadOrders(1), 500));
            $('#orderStatusFilter').change(() => loadOrders(1));
            $('#syncOrdersBtn').click(syncOrders);
            $('#fullSyncOrders').click(fullSyncOrders);

            // صفحه شناسنامه
            $('#certSearch').on('input', debounce(() => loadCertificates(1), 500));
            $('#certStatusFilter').change(() => loadCertificates(1));
            $('#certDateFrom, #certDateTo').change(() => loadCertificates(1));
            $('#certProductSearch').on('input', debounce(searchProductForCertificate, 500));

            // صفحه لاگ‌ها
            $('[data-log-type]').click(function() {
                const logType = $(this).data('log-type');
                switchLogType(logType);
            });

            // صفحه تنظیمات
            $('#testConnection').click(testWooConnection);
            $('#saveWooSettings').click(saveWooSettings);
            $('#saveAppearance').click(saveAppearance);
        });

        function updateDateTime() {
            const now = new Date();
            const timeString = now.toLocaleTimeString('fa-IR', { hour: '2-digit', minute: '2-digit' });
            $('#currentTime').text(timeString);
        }

        function loadCurrentPage() {
            switch(appState.currentPage) {
                case 'dashboard': loadDashboard(); break;
                case 'products': loadProducts(); loadCategories(); loadAttributes(); break;
                case 'orders': loadOrders(); break;
                case 'stranger-order': resetStrangerOrderForm(); break;
                case 'certificates': loadCertificates(); loadCertificateStats(); break;
                case 'issue-certificate': break;
                case 'logs': loadSystemLogs(); break;
                case 'settings': loadSettings(); break;
            }
        }

        // ========== داشبورد ==========
        function loadDashboard() {
            $.get('?api=1&action=get_comprehensive_stats', function(response) {
                if(response.success) {
                    const stats = response.stats;
                    $('#totalProducts').text(stats.total_products.toLocaleString());
                    $('#productsValue').text(stats.total_products_value.toLocaleString() + ' تومان');
                    $('#totalCustomers').text(stats.total_customers.toLocaleString());
                    $('#totalOrders').text(stats.total_orders.toLocaleString());
                    $('#totalRevenue').text(stats.total_revenue.toLocaleString() + ' تومان');
                    $('#stockStatus').text((stats.low_stock + stats.out_of_stock).toLocaleString());
                    $('#stockDetails').text(stats.low_stock.toLocaleString() + ' کمبود، ' + stats.out_of_stock.toLocaleString() + ' ناموجود');
                    $('#totalCertificates').text(stats.total_certificates.toLocaleString());
                    $('#activeCertificates').text(stats.active_certificates.toLocaleString());

                    let topProductsHtml = '';
                    response.top_products.forEach(p => {
                        topProductsHtml += `<tr><td>${p.name}</td><td>${p.sku}</td><td>${p.total_quantity}</td><td>${p.stock}</td></tr>`;
                    });
                    $('#topProductsTable').html(topProductsHtml);

                    let topCustomersHtml = '';
                    response.top_customers.forEach(c => {
                        topCustomersHtml += `<tr><td>${c.fullname}</td><td>${c.phone}</td><td>${c.order_count}</td><td>${parseInt(c.total_spent).toLocaleString()} تومان</td></tr>`;
                    });
                    $('#topCustomersTable').html(topCustomersHtml);

                    createSalesChart();
                    createTopProductsChart(response.top_products);
                }
            });
        }

        function createSalesChart() {
            // پیاده‌سازی نمودار فروش (مشابه قبل)
        }

        function createTopProductsChart(products) {
            // پیاده‌سازی نمودار محصولات پرفروش
        }

        // ========== محصولات ==========
        function loadProducts(page = 1) {
            const search = $('#productSearch').val();
            const category = $('#productCategoryFilter').val();
            const attribute = $('#productAttributeFilter').val();
            const sort = appState.productSort.field;
            const order = appState.productSort.order;
            appState.currentProductPage = page;

            $.get(`?api=1&action=get_products&page=${page}&search=${encodeURIComponent(search)}&category=${encodeURIComponent(category)}&attribute=${encodeURIComponent(attribute)}&sort=${sort}&order=${order}`, function(response) {
                if(response.success) {
                    let html = '';
                    response.products.forEach(p => {
                        html += `<tr>
                            <td>${p.name}</td>
                            <td>${parseInt(p.price).toLocaleString()} تومان</td>
                            <td>${p.price_per_gram ? p.price_per_gram.toLocaleString() : '-'} تومان</td>
                            <td><span class="badge ${p.stock > 10 ? 'badge-success' : p.stock > 0 ? 'badge-warning' : 'badge-danger'}">${p.stock}</span></td>
                            <td>${p.category}</td>
                            <td>
                                <button class="btn btn-sm btn-secondary" onclick="viewProduct(${p.id})">مشاهده</button>
                                <button class="btn btn-sm btn-primary" onclick="quickEditProduct(${p.id})">ویرایش</button>
                                <button class="btn btn-sm btn-info" onclick="showCertificate(${p.id})">شناسنامه</button>
                            </td>
                        </tr>`;
                    });
                    $('#productsTable tbody').html(html);
                    createPagination('productsPagination', response.pagination, loadProducts);
                }
            });
        }

        function loadCategories() {
            $.get('?api=1&action=get_categories', function(response) {
                if(response.success) {
                    let html = '<option value="">همه دسته‌ها</option>';
                    response.categories.forEach(c => {
                        html += `<option value="${c.category}">${c.category} (${c.product_count})</option>`;
                    });
                    $('#productCategoryFilter').html(html);
                    $('#bulkCategory').html(html);
                }
            });
        }

        function loadAttributes() {
            $.get('?api=1&action=get_attributes', function(response) {
                if(response.success) {
                    let html = '<option value="">همه ویژگی‌ها</option>';
                    Object.keys(response.attributes).forEach(attr => {
                        html += `<option value="${attr}">${attr}</option>`;
                    });
                    $('#productAttributeFilter').html(html);
                    $('#bulkAttribute').html(html);
                }
            });
        }

        function viewProduct(id) {
            $.get(`?api=1&action=get_product&id=${id}`, function(response) {
                if(response.success) {
                    const p = response.product;
                    let html = `<div class="row">...`; // جزئیات محصول
                    $('#productModalContent').html(html);
                    $('#productModal').show();
                }
            });
        }

        function quickEditProduct(id) {
            viewProduct(id); // همان تابع برای ویرایش سریع
        }

        function showCertificate(productId) {
            $.get(`?api=1&action=generate_certificate&product_id=${productId}`, function(response) {
                if(response.success) {
                    const cert = response.certificate;
                    const html = generateCertificateHTML(cert);
                    $('#certificateModalContent').html(html);
                    $('#certificateModal').show();
                }
            });
        }

        function applyBulkPricePerGram() {
            const category = $('#bulkCategory').val();
            const attribute = $('#bulkAttribute').val();
            const pricePerGram = parseFloat($('#bulkPricePerGram').val());
            const limit = parseInt($('#bulkLimit').val()) || 0;
            if(!pricePerGram) {
                alert('لطفاً قیمت هر گرم را وارد کنید');
                return;
            }
            // اینجا باید API جداگانه پیاده‌سازی شود
            alert('این قابلیت در حال پیاده‌سازی است');
        }

        function syncProducts() {
            $('#syncProductsBtn').prop('disabled', true).text('در حال همگام‌سازی...');
            $.post('?api=1&action=sync_products', {page: 1}, function(response) {
                if(response.success) {
                    alert('همگام‌سازی انجام شد');
                    loadProducts();
                } else {
                    alert('خطا: ' + response.message);
                }
                $('#syncProductsBtn').prop('disabled', false).text('همگام‌سازی');
            });
        }

        function fullSyncProducts() {
            let page = 1;
            let total = 0;
            function syncPage() {
                $.post('?api=1&action=sync_products', {page: page}, function(response) {
                    if(response.success) {
                        total += response.result.synced;
                        if(response.result.has_more) {
                            page++;
                            syncPage();
                        } else {
                            alert(`همگام‌سازی کامل شد. ${total} محصول سینک شد.`);
                            loadProducts();
                        }
                    } else {
                        alert('خطا: ' + response.message);
                    }
                });
            }
            syncPage();
        }

        // ========== سفارشات ==========
        function loadOrders(page = 1) {
            const search = $('#orderSearch').val();
            const status = $('#orderStatusFilter').val();
            appState.currentOrderPage = page;

            $.get(`?api=1&action=get_orders&page=${page}&search=${encodeURIComponent(search)}&status=${status}`, function(response) {
                if(response.success) {
                    let html = '';
                    response.orders.forEach(o => {
                        const statusMap = {
                            'pending': 'در انتظار', 'processing': 'در حال پردازش', 'completed': 'تکمیل شده',
                            'cancelled': 'لغو شده', 'stranger': 'بیگانگان'
                        };
                        html += `<tr>
                            <td>${o.order_number}</td>
                            <td>${o.customer_name || 'ناشناس'}</td>
                            <td>${parseInt(o.total_amount).toLocaleString()} تومان</td>
                            <td>${statusMap[o.status] || o.status}</td>
                            <td>${new Date(o.created_at).toLocaleDateString('fa-IR')}</td>
                            <td>
                                <button class="btn btn-sm btn-secondary" onclick="viewOrder(${o.id})">مشاهده</button>
                                <button class="btn btn-sm btn-primary" onclick="generateLabel(${o.id})">برچسب</button>
                            </td>
                        </tr>`;
                    });
                    $('#ordersTable').html(html);
                    createPagination('ordersPagination', response.pagination, loadOrders);
                }
            });
        }

        function viewOrder(id) {
            $.get(`?api=1&action=get_order&id=${id}`, function(response) {
                if(response.success) {
                    // نمایش جزئیات در مودال
                }
            });
        }

        function generateLabel(id) {
            $.get(`?api=1&action=get_order&id=${id}`, function(response) {
                if(response.success) {
                    const o = response.order;
                    const label = `
                        <div class="label-preview">
                            <div class="label-header"><span class="label-name">${o.customer_name}</span><span class="label-insurance">بیمه</span></div>
                            <div class="label-address">${o.shipping_address.address_1 || ''}</div>
                            <div class="label-contact"><span>تلفن: ${o.customer_phone}</span><span>کدپستی: ${o.shipping_address.postcode || ''}</span></div>
                            <div class="label-notes">یادداشت: ${o.notes || ''}</div>
                            <div class="label-meta"><span class="label-meta-item">سفارش: ${o.order_number}</span></div>
                            <div class="label-footer">mashahir.jewelry</div>
                        </div>
                    `;
                    $('#labelPreview').html(label);
                    $('#labelModal').show();
                }
            });
        }

        function printLabel() {
            window.print();
        }

        function syncOrders() {
            $('#syncOrdersBtn').prop('disabled', true).text('در حال همگام‌سازی...');
            $.post('?api=1&action=sync_orders', {page: 1}, function(response) {
                if(response.success) {
                    alert('همگام‌سازی انجام شد');
                    loadOrders();
                } else {
                    alert('خطا: ' + response.message);
                }
                $('#syncOrdersBtn').prop('disabled', false).text('همگام‌سازی');
            });
        }

        function fullSyncOrders() {
            let page = 1;
            let total = 0;
            function syncPage() {
                $.post('?api=1&action=sync_orders', {page: page}, function(response) {
                    if(response.success) {
                        total += response.result.synced;
                        if(response.result.has_more) {
                            page++;
                            syncPage();
                        } else {
                            alert(`همگام‌سازی کامل شد. ${total} سفارش سینک شد.`);
                            loadOrders();
                        }
                    } else {
                        alert('خطا: ' + response.message);
                    }
                });
            }
            syncPage();
        }

        // ========== سفارش بیگانگان ==========
        function searchCustomer() {
            const phone = $('#customerPhoneSearch').val();
            if(phone.length < 5) {
                $('#customerSearchResults').hide();
                return;
            }

            $.get(`?api=1&action=search_customer_by_phone&phone=${encodeURIComponent(phone)}`, function(response) {
                if(response.success) {
                    const c = response.customer;
                    $('#strangerCustomerName').val(c.fullname);
                    $('#strangerCustomerPhone').val(c.phone);
                    $('#strangerCustomerAddress').val(c.address || '');
                    $('#strangerCustomerCity').val(c.city || '');
                    $('#strangerCustomerProvince').val(c.province || '');
                    $('#strangerCustomerPostal').val(c.postal_code || '');
                } else {
                    $('#strangerCustomerPhone').val(phone);
                }
            });
        }

        function searchProductForStranger() {
            const query = $('#strangerProductSearch').val();
            if(query.length < 2) {
                $('#productSearchResults').hide();
                return;
            }

            $.get(`?api=1&action=search_product_by_sku&sku=${encodeURIComponent(query)}`, function(response) {
                if(response.success && response.products.length > 0) {
                    let html = '';
                    response.products.forEach(p => {
                        html += `<div class="search-result-item" onclick="addProductToStrangerOrder({id: ${p.id}, name: '${p.name}', price: ${p.price}, sku: '${p.sku}'})">${p.name} - ${p.sku} - ${p.price.toLocaleString()} تومان</div>`;
                    });
                    $('#productSearchResults').html(html).show();
                } else {
                    $('#productSearchResults').html(`<div class="search-result-item" onclick="addManualProductToStrangerOrder('${query}')">افزودن دستی: "${query}"</div>`).show();
                }
            });
        }

        function addProductToStrangerOrder(product) {
            $('#productSearchResults').hide();
            $('#strangerProductSearch').val('');
            const existing = appState.strangerOrderItems.find(i => i.id === product.id);
            if(existing) {
                existing.quantity++;
            } else {
                appState.strangerOrderItems.push({...product, quantity: 1});
            }
            updateStrangerOrderDisplay();
        }

        function addManualProductToStrangerOrder(query) {
            $('#productSearchResults').hide();
            $('#strangerProductSearch').val('');
            const parts = query.split(' - ');
            const name = parts[0] || query;
            const price = parts.length > 1 ? parseFloat(parts[1]) : 0;
            appState.strangerOrderItems.push({id: 0, name, price, quantity: 1, sku: ''});
            updateStrangerOrderDisplay();
        }

        function updateStrangerOrderDisplay() {
            let html = '';
            let total = 0;
            appState.strangerOrderItems.forEach((item, index) => {
                total += item.price * item.quantity;
                html += `<div class="order-item">
                    <div><strong>${item.name}</strong><br><small>${item.price.toLocaleString()} تومان</small></div>
                    <div class="d-flex gap-2">
                        <button class="btn btn-sm btn-secondary" onclick="updateStrangerItemQty(${index}, -1)">-</button>
                        <span>${item.quantity}</span>
                        <button class="btn btn-sm btn-secondary" onclick="updateStrangerItemQty(${index}, 1)">+</button>
                        <button class="btn btn-sm btn-danger" onclick="removeStrangerItem(${index})">حذف</button>
                    </div>
                </div>`;
            });
            $('#strangerOrderItems').html(html);
            $('#strangerOrderTotal').text(total.toLocaleString());
        }

        function updateStrangerItemQty(index, change) {
            const item = appState.strangerOrderItems[index];
            item.quantity += change;
            if(item.quantity <= 0) appState.strangerOrderItems.splice(index, 1);
            updateStrangerOrderDisplay();
        }

        function removeStrangerItem(index) {
            appState.strangerOrderItems.splice(index, 1);
            updateStrangerOrderDisplay();
        }

        function createStrangerOrder() {
            if(appState.strangerOrderItems.length === 0) {
                alert('حداقل یک محصول اضافه کنید');
                return;
            }

            const data = {
                customer_name: $('#strangerCustomerName').val(),
                customer_phone: $('#strangerCustomerPhone').val(),
                customer_address: $('#strangerCustomerAddress').val(),
                customer_city: $('#strangerCustomerCity').val(),
                customer_province: $('#strangerCustomerProvince').val(),
                customer_postal: $('#strangerCustomerPostal').val(),
                notes: $('#strangerCustomerNotes').val(),
                items: appState.strangerOrderItems,
                total: appState.strangerOrderItems.reduce((s, i) => s + i.price * i.quantity, 0)
            };

            $.post('?api=1&action=create_customer_order', data, function(response) {
                if(response.success) {
                    alert('سفارش با موفقیت ثبت شد');
                    resetStrangerOrderForm();
                } else {
                    alert('خطا: ' + response.message);
                }
            }).fail(() => alert('خطا در ارتباط'));
        }

        function resetStrangerOrderForm() {
            appState.strangerOrderItems = [];
            $('#strangerCustomerName, #strangerCustomerPhone, #strangerCustomerAddress, #strangerCustomerCity, #strangerCustomerProvince, #strangerCustomerPostal, #strangerCustomerNotes').val('');
            $('#strangerOrderItems').empty();
            $('#strangerOrderTotal').text('0');
        }

        // ========== شناسنامه ==========
        function loadCertificates(page = 1) {
            const search = $('#certSearch').val();
            const status = $('#certStatusFilter').val();
            const dateFrom = $('#certDateFrom').val();
            const dateTo = $('#certDateTo').val();
            appState.currentCertPage = page;

            $.get(`?api=1&action=get_certificates&page=${page}&search=${encodeURIComponent(search)}&status=${status}&date_from=${dateFrom}&date_to=${dateTo}`, function(response) {
                if(response.success) {
                    let html = '';
                    response.certificates.forEach(c => {
                        const statusClass = c.status === 'active' ? 'badge-success' : 'badge-danger';
                        html += `<tr>
                            <td><img src="${c.image_url || 'https://via.placeholder.com/50'}" style="width:50px;height:50px;object-fit:cover;border-radius:5px;"></td>
                            <td>${c.product_name}</td>
                            <td>${c.product_data.category || '-'}</td>
                            <td>${new Date(c.issue_date).toLocaleDateString('fa-IR')}</td>
                            <td>${c.short_code}</td>
                            <td>${c.certificate_number}</td>
                            <td><span class="badge ${statusClass}">${c.status === 'active' ? 'فعال' : 'باطل شده'}</span></td>
                            <td>
                                <button class="btn btn-sm btn-secondary" onclick="viewCertificate(${c.id})">نمایش</button>
                                <button class="btn btn-sm btn-danger" onclick="revokeCertificate(${c.id})">ابطال</button>
                            </td>
                        </tr>`;
                    });
                    $('#certificatesTable').html(html);
                    createPagination('certificatesPagination', response.pagination, loadCertificates);
                }
            });
        }

        function loadCertificateStats() {
            $.get('?api=1&action=get_comprehensive_stats', function(response) {
                if(response.success) {
                    $('#totalCertificates').text(response.stats.total_certificates.toLocaleString());
                    $('#todayCertificates').text(response.stats.today_certificates.toLocaleString());
                    $('#activeCertificates').text(response.stats.active_certificates.toLocaleString());
                    $('#revokedCertificates').text(response.stats.revoked_certificates.toLocaleString());
                }
            });
        }

        function searchProductForCertificate() {
            const query = $('#certProductSearch').val();
            if(query.length < 2) {
                $('#certProductResults').hide();
                return;
            }

            $.get(`?api=1&action=search_products_for_certificate&q=${encodeURIComponent(query)}`, function(response) {
                if(response.success && response.products.length > 0) {
                    let html = '';
                    response.products.forEach(p => {
                        html += `<div class="search-result-item" onclick="selectProductForCertificate(${p.id}, '${p.name}', '${p.sku}', ${p.weight || 0}, ${p.length || 0}, ${p.width || 0}, ${p.height || 0}, '${p.images ? p.images[0] || '' : ''}')">${p.name} - ${p.sku}</div>`;
                    });
                    $('#certProductResults').html(html).show();
                } else {
                    $('#certProductResults').html('<div class="search-result-item">نتیجه‌ای یافت نشد</div>').show();
                }
            });
        }

        function selectProductForCertificate(id, name, sku, weight, length, width, height, image) {
            $('#certProductId').val(id);
            $('#certProductName').val(name);
            $('#certProductSku').val(sku);
            $('#certWeight').val(weight);
            $('#certLength').val(length);
            $('#certWidth').val(width);
            $('#certHeight').val(height);
            $('#certImageUrl').val(image);
            $('#certProductResults').hide();
            $('#certProductSearch').val('');
        }

        function previewCertificate() {
            const name = $('#certProductName').val();
            const sku = $('#certProductSku').val();
            const weight = $('#certWeight').val();
            const metal = $('#certMetal').val();
            const stone = $('#certStoneAuthenticity').val();
            const image = $('#certImageUrl').val() || 'https://via.placeholder.com/250';
            const html = `
                <div class="certificate-card" style="margin:0 auto;">
                    <div class="security-pattern"></div>
                    <div class="watermark"></div>
                    <div class="gold-bar"></div>
                    <div class="certificate-header">
                        <div class="certificate-title">CERTIFICATE</div>
                        <div class="certificate-subtitle">Mashahir Jewelry Authentication</div>
                    </div>
                    <div class="certificate-content">
                        <div class="certificate-info">
                            <div class="certificate-row"><span class="certificate-label">Stone</span><span class="certificate-value">${stone}</span></div>
                            <div class="certificate-row"><span class="certificate-label">Metal</span><span class="certificate-value">${metal}</span></div>
                            <div class="certificate-row"><span class="certificate-label">Weight</span><span class="certificate-value">${weight} g</span></div>
                            <div class="certificate-row"><span class="certificate-label">SKU</span><span class="certificate-value">${sku}</span></div>
                        </div>
                        <div class="certificate-image-box"><img src="${image}" style="width:100%;height:100%;object-fit:cover;"></div>
                    </div>
                    <div class="certificate-footer">
                        <div class="certificate-qr"><img src="https://chart.googleapis.com/chart?chs=100x100&cht=qr&chl=${sku}&choe=UTF-8"></div>
                    </div>
                    <div class="certificate-microtext">MASHAHIR•AUTHENTIC•JEWELRY•ORIGINAL•</div>
                    <div class="certificate-secret">CERT-${sku}</div>
                </div>
            `;
            $('#certificatePreview').html(html);
            $('#certificatePreviewArea').show();
        }

        function issueCertificate() {
            const data = {
                product_id: $('#certProductId').val(),
                product_sku: $('#certProductSku').val(),
                product_name: $('#certProductName').val(),
                attributes: {
                    weight: $('#certWeight').val(),
                    dimensions: {
                        length: $('#certLength').val(),
                        width: $('#certWidth').val(),
                        height: $('#certHeight').val()
                    },
                    metal: $('#certMetal').val(),
                    stone: $('#certStoneAuthenticity').val()
                },
                image_url: $('#certImageUrl').val(),
                issue_date: new Date().toISOString().split('T')[0]
            };

            $.post('?api=1&action=issue_certificate', data, function(response) {
                if(response.success) {
                    alert(`شناسنامه با شماره ${response.certificate.number} صادر شد`);
                    window.location.href = '?page=certificates';
                } else {
                    alert('خطا: ' + response.message);
                }
            });
        }

        function viewCertificate(id) {
            $.get(`?api=1&action=get_certificate&id=${id}`, function(response) {
                if(response.success) {
                    const c = response.certificate;
                    const html = generateCertificateHTML(c);
                    $('#certificateModalContent').html(html);
                    $('#certificateModal').show();
                }
            });
        }

        function generateCertificateHTML(cert) {
            const product = cert.product_data;
            const image = cert.image_url || (product.images ? product.images[0] : 'https://via.placeholder.com/250');
            return `
                <div class="certificate-card" style="margin:0 auto;">
                    <div class="security-pattern"></div>
                    <div class="watermark"></div>
                    <div class="gold-bar"></div>
                    <div class="certificate-header">
                        <div class="certificate-title">CERTIFICATE</div>
                        <div class="certificate-subtitle">Mashahir Jewelry Authentication</div>
                    </div>
                    <div class="certificate-content">
                        <div class="certificate-info">
                            <div class="certificate-row"><span class="certificate-label">Product</span><span class="certificate-value">${cert.product_name}</span></div>
                            <div class="certificate-row"><span class="certificate-label">Stone</span><span class="certificate-value">${cert.attributes.stone || ''}</span></div>
                            <div class="certificate-row"><span class="certificate-label">Metal</span><span class="certificate-value">${cert.attributes.metal || ''}</span></div>
                            <div class="certificate-row"><span class="certificate-label">Weight</span><span class="certificate-value">${cert.attributes.weight || ''} g</span></div>
                            <div class="certificate-row"><span class="certificate-label">Code</span><span class="certificate-value">${cert.short_code}</span></div>
                        </div>
                        <div class="certificate-image-box"><img src="${image}" style="width:100%;height:100%;object-fit:cover;"></div>
                    </div>
                    <div class="certificate-footer">
                        <div class="certificate-qr"><img src="https://chart.googleapis.com/chart?chs=100x100&cht=qr&chl=${cert.short_code}&choe=UTF-8"></div>
                    </div>
                    <div class="certificate-microtext">MASHAHIR•AUTHENTIC•JEWELRY•ORIGINAL•</div>
                    <div class="certificate-secret">${cert.certificate_number}</div>
                </div>
            `;
        }

        function revokeCertificate(id) {
            if(!confirm('آیا از ابطال این شناسنامه اطمینان دارید؟')) return;
            $.post('?api=1&action=revoke_certificate', {id}, function(response) {
                if(response.success) {
                    alert('شناسنامه باطل شد');
                    loadCertificates(appState.currentCertPage);
                } else {
                    alert('خطا: ' + response.message);
                }
            });
        }

        function printCertificate() {
            window.print();
        }

        function downloadCertificate() {
            // تابع دانلود PDF (قابل توسعه)
        }

        // ========== لاگ‌ها ==========
        function switchLogType(type) {
            if(type === 'system') {
                $('#systemLogsSection').removeClass('d-none');
                $('#productLogsSection').addClass('d-none');
                loadSystemLogs();
            } else {
                $('#systemLogsSection').addClass('d-none');
                $('#productLogsSection').removeClass('d-none');
                loadProductLogs();
            }
        }

        function loadSystemLogs(page = 1) {
            appState.currentSystemLogPage = page;
            $.get(`?api=1&action=get_logs&page=${page}`, function(response) {
                if(response.success) {
                    let html = '';
                    response.logs.forEach(l => {
                        html += `<tr><td>${new Date(l.created_at).toLocaleString('fa-IR')}</td><td>${l.action}</td><td>${l.description}</td><td>${l.user_name || 'سیستم'}</td></tr>`;
                    });
                    $('#systemLogsTable').html(html);
                    createPagination('systemLogsPagination', response.pagination, loadSystemLogs);
                }
            });
        }

        function loadProductLogs(page = 1) {
            appState.currentProductLogPage = page;
            $.get(`?api=1&action=get_product_logs&page=${page}`, function(response) {
                if(response.success) {
                    let html = '';
                    response.logs.forEach(l => {
                        html += `<tr><td>${new Date(l.created_at).toLocaleString('fa-IR')}</td><td>${l.product_name}</td><td>${l.action}</td><td>${l.changed_fields.join(', ')}</td><td>${l.user_name || 'سیستم'}</td></tr>`;
                    });
                    $('#productLogsTable').html(html);
                    createPagination('productLogsPagination', response.pagination, loadProductLogs);
                }
            });
        }

        // ========== تنظیمات ==========
        function loadSettings() {
            $.get('?api=1&action=get_products&page=1&per_page=1', function() {});
            const primaryColor = localStorage.getItem('shopgun_primary_color') || '#40E0D0';
            const theme = localStorage.getItem('shopgun_theme') || 'dark';
            $('#primaryColor').val(primaryColor);
            $('#themeSelect').val(theme);
            applyAppearanceSettings();
        }

        function testWooConnection() {
            $.post('?api=1&action=test_connection', function(response) {
                alert(response.success ? 'اتصال موفق' : 'خطا: ' + response.message);
            });
        }

        function saveWooSettings() {
            const settings = {
                'woocommerce_store_url': $('#storeUrl').val(),
                'woocommerce_consumer_key': $('#consumerKey').val(),
                'woocommerce_consumer_secret': $('#consumerSecret').val()
            };
            let saved = 0;
            Object.keys(settings).forEach(key => {
                $.post('?api=1&action=save_settings', {key, value: settings[key]}, function() {
                    saved++;
                    if(saved === Object.keys(settings).length) alert('تنظیمات ذخیره شد');
                });
            });
        }

        function saveAppearance() {
            localStorage.setItem('shopgun_primary_color', $('#primaryColor').val());
            localStorage.setItem('shopgun_theme', $('#themeSelect').val());
            applyAppearanceSettings();
        }

        function applyAppearanceSettings() {
            const primaryColor = localStorage.getItem('shopgun_primary_color') || '#40E0D0';
            const theme = localStorage.getItem('shopgun_theme') || 'dark';
            document.documentElement.style.setProperty('--accent-turquoise', primaryColor);
            if(theme === 'light') {
                document.body.classList.add('light-theme');
            } else {
                document.body.classList.remove('light-theme');
            }
        }

        // ========== ابزارهای کمکی ==========
        function debounce(func, wait) {
            let timeout;
            return function(...args) {
                clearTimeout(timeout);
                timeout = setTimeout(() => func.apply(this, args), wait);
            };
        }

        function createPagination(containerId, pagination, callback) {
            const { page, total_pages } = pagination;
            let html = '';
            if(page > 1) html += `<div class="page-item" onclick="${callback.name}(${page-1})">قبلی</div>`;
            for(let i = Math.max(1, page-2); i <= Math.min(total_pages, page+2); i++) {
                html += `<div class="page-item ${i===page?'active':''}" onclick="${callback.name}(${i})">${i}</div>`;
            }
            if(page < total_pages) html += `<div class="page-item" onclick="${callback.name}(${page+1})">بعدی</div>`;
            $(`#${containerId}`).html(html);
        }
    </script>
</body>
</html>
