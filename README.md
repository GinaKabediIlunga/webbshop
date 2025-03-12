import java.sql.*;
import java.util.Scanner;

public class ECommerceBackend {
    private static final String URL = "jdbc:sqlite:ecommerce.db";

    public static void main(String[] args) {
        createTables();
        insertInitialData();
        displayMenu();
    }

    private static void createTables() {
        try (Connection conn = DriverManager.getConnection(URL);
             Statement stmt = conn.createStatement()) {
            
            String createCustomersTable = "CREATE TABLE IF NOT EXISTS Customers (" +
                    "customer_id INTEGER PRIMARY KEY AUTOINCREMENT, " +
                    "name TEXT NOT NULL, " +
                    "email TEXT UNIQUE NOT NULL, " +
                    "password TEXT NOT NULL, " +
                    "address TEXT NOT NULL);";

            String createProductsTable = "CREATE TABLE IF NOT EXISTS Products (" +
                    "product_id INTEGER PRIMARY KEY AUTOINCREMENT, " +
                    "model TEXT NOT NULL, " +
                    "category TEXT NOT NULL, " +
                    "price INTEGER NOT NULL, " +
                    "stock_quantity INTEGER NOT NULL);";

            String createOrdersTable = "CREATE TABLE IF NOT EXISTS Orders (" +
                    "order_id INTEGER PRIMARY KEY AUTOINCREMENT, " +
                    "customer_id INTEGER NOT NULL, " +
                    "order_date TIMESTAMP DEFAULT CURRENT_TIMESTAMP, " +
                    "total_price INTEGER NOT NULL, " +
                    "insurance BOOLEAN DEFAULT 0, " +
                    "FOREIGN KEY (customer_id) REFERENCES Customers(customer_id) ON DELETE CASCADE);";

            stmt.execute(createCustomersTable);
            stmt.execute(createProductsTable);
            stmt.execute(createOrdersTable);
            System.out.println("Databas och tabeller skapade.");
        } catch (SQLException e) {
            e.printStackTrace();
        }
    }

    private static void insertInitialData() {
        try (Connection conn = DriverManager.getConnection(URL);
             Statement stmt = conn.createStatement()) {
            
            String insertProducts = "INSERT INTO Products (model, category, price, stock_quantity) VALUES " +
                    "('Volvo XC90', 'SUV', 690000, 10)," +
                    "('Volvo XC60', 'SUV', 555000, 15)," +
                    "('Volvo V90', 'Sedan', 635000, 8)";
            
            stmt.execute(insertProducts);
            System.out.println("Produkter har lagts till i databasen.");
        } catch (SQLException e) {
            e.printStackTrace();
        }
    }

    private static void displayMenu() {
        Scanner scanner = new Scanner(System.in);
        while (true) {
            System.out.println("\n1. Lista alla produkter");
            System.out.println("2. Lägg till en order");
            System.out.println("3. Visa orderhistorik");
            System.out.println("4. Avsluta");
            System.out.print("Välj ett alternativ: ");

            int choice = scanner.nextInt();
            switch (choice) {
                case 1:
                    listAllProducts();
                    break;
                case 2:
                    placeOrder();
                    break;
                case 3:
                    showOrderHistory();
                    break;
                case 4:
                    System.out.println("Avslutar...");
                    scanner.close();
                    return;
                default:
                    System.out.println("Ogiltigt val. Försök igen.");
            }
        }
    }

    private static void listAllProducts() {
        try (Connection conn = DriverManager.getConnection(URL);
             Statement stmt = conn.createStatement();
             ResultSet rs = stmt.executeQuery("SELECT * FROM Products")) {
            
            System.out.println("\nTillgängliga produkter:");
            while (rs.next()) {
                System.out.println(rs.getInt("product_id") + " - " + rs.getString("model") + " - " + rs.getInt("price") + " SEK");
            }
        } catch (SQLException e) {
            e.printStackTrace();
        }
    }

    private static void placeOrder() {
        try (Connection conn = DriverManager.getConnection(URL);
             Scanner scanner = new Scanner(System.in)) {
            
            System.out.print("Ange kund-ID: ");
            int customerId = scanner.nextInt();
            
            System.out.print("Ange produkt-ID: ");
            int productId = scanner.nextInt();
            
            System.out.print("Ange antal: ");
            int quantity = scanner.nextInt();
            
            String getProductQuery = "SELECT price FROM Products WHERE product_id = ?";
            try (PreparedStatement pstmt = conn.prepareStatement(getProductQuery)) {
                pstmt.setInt(1, productId);
                ResultSet rs = pstmt.executeQuery();
                if (rs.next()) {
                    int price = rs.getInt("price");
                    int totalPrice = price * quantity;
                    
                    String insertOrderQuery = "INSERT INTO Orders (customer_id, total_price) VALUES (?, ?)";
                    try (PreparedStatement orderStmt = conn.prepareStatement(insertOrderQuery)) {
                        orderStmt.setInt(1, customerId);
                        orderStmt.setInt(2, totalPrice);
                        orderStmt.executeUpdate();
                        System.out.println("Order skapad för kund " + customerId + " på " + totalPrice + " SEK");
                    }
                } else {
                    System.out.println("Produkten hittades inte.");
                }
            }
        } catch (SQLException e) {
            e.printStackTrace();
        }
    }

    private static void showOrderHistory() {
        try (Connection conn = DriverManager.getConnection(URL);
             Statement stmt = conn.createStatement();
             ResultSet rs = stmt.executeQuery("SELECT * FROM Orders")) {
            
            System.out.println("\nOrderhistorik:");
            while (rs.next()) {
                System.out.println("Order ID: " + rs.getInt("order_id") + ", Kund ID: " + rs.getInt("customer_id") + ", Pris: " + rs.getInt("total_price") + " SEK");
            }
        } catch (SQLException e) {
            e.printStackTrace();
        }
    }
}
