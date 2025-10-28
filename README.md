# Library-Management-System
Developed a console-based application to manage books in a library. Implemented features like adding books, issuing books, and returning books using Java and MySQL (via JDBC).
import java.sql.*;
import java.util.Scanner;

public class LibraryManagement {
    private static final String URL = "jdbc:postgresql://localhost:5432/librarydb";
    private static final String USER = "postgres";
    private static final String PASSWORD = "Yashvin@19";

    public static void main(String[] args) {
        Scanner sc = new Scanner(System.in);
        try (Connection conn = DriverManager.getConnection(URL, USER, PASSWORD)) {
            while (true) {
                System.out.println("\n=== Library Management System ===");
                System.out.println("1. Add Book");
                System.out.println("2. Issue Book");
                System.out.println("3. Return Book");
                System.out.println("4. View All Books");
                System.out.println("5. Exit");
                System.out.print("Choose an option: ");
                int choice = sc.nextInt();
                sc.nextLine();

                switch (choice) {
                    case 1 -> addBook(conn, sc);
                    case 2 -> issueBook(conn, sc);
                    case 3 -> returnBook(conn, sc);
                    case 4 -> viewBooks(conn);
                    case 5 -> {
                        System.out.println("Exiting...");
                        return;
                    }
                    default -> System.out.println("Invalid choice!");
                }
            }
        } catch (SQLException e) {
            e.printStackTrace();
        }
    }

    private static void addBook(Connection conn, Scanner sc) throws SQLException {
        System.out.print("Enter book title: ");
        String title = sc.nextLine();
        System.out.print("Enter author name: ");
        String author = sc.nextLine();
        String sql = "INSERT INTO books (title, author) VALUES (?, ?)";
        try (PreparedStatement stmt = conn.prepareStatement(sql)) {
            stmt.setString(1, title);
            stmt.setString(2, author);
            stmt.executeUpdate();
            System.out.println("Book added successfully!");
        }
    }

    private static void issueBook(Connection conn, Scanner sc) throws SQLException {
        System.out.print("Enter book ID to issue: ");
        int id = sc.nextInt();
        String checkSql = "SELECT is_issued FROM books WHERE id = ?";
        try (PreparedStatement checkStmt = conn.prepareStatement(checkSql)) {
            checkStmt.setInt(1, id);
            ResultSet rs = checkStmt.executeQuery();
            if (rs.next()) {
                if (rs.getBoolean("is_issued")) {
                    System.out.println("Book is already issued!");
                    return;
                }
            } else {
                System.out.println("Book not found!");
                return;
            }
        }
        String sql = "UPDATE books SET is_issued = TRUE WHERE id = ?";
        try (PreparedStatement stmt = conn.prepareStatement(sql)) {
            stmt.setInt(1, id);
            int rows = stmt.executeUpdate();
            if (rows > 0) System.out.println("Book issued successfully!");
            else System.out.println("Failed to issue book.");
        }
    }

    private static void returnBook(Connection conn, Scanner sc) throws SQLException {
        System.out.print("Enter book ID to return: ");
        int id = sc.nextInt();
        String sql = "UPDATE books SET is_issued = FALSE WHERE id = ?";
        try (PreparedStatement stmt = conn.prepareStatement(sql)) {
            stmt.setInt(1, id);
            int rows = stmt.executeUpdate();
            if (rows > 0) System.out.println("Book returned successfully!");
            else System.out.println("Book not found!");
        }
    }

    private static void viewBooks(Connection conn) throws SQLException {
        String sql = "SELECT * FROM books ORDER BY id";
        try (Statement stmt = conn.createStatement();
             ResultSet rs = stmt.executeQuery(sql)) {
            System.out.println("\n--- Books in Library ---");
            while (rs.next()) {
                System.out.printf("ID: %d | Title: %s | Author: %s | Issued: %s%n",
                        rs.getInt("id"),
                        rs.getString("title"),
                        rs.getString("author"),
                        rs.getBoolean("is_issued") ? "Yes" : "No");
            }
        }
    }
}
