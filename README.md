# programacion-2-Projecto-final
import java.sql.*;
import java.util.ArrayList;
import java.util.List;
import java.util.Scanner;

// Clase para conexión a la base de datos
class DBConnection {
    private static final String URL = "jdbc:mysql://127.0.0.1:3306/sakila2";
    private static final String USER = "root";
    private static final String PASSWORD = "1234";

    // Método para obtener conexión a la base de datos
    public static Connection getConnection() throws SQLException {
        try {
            // Intentar cargar el driver
            Class.forName("com.mysql.cj.jdbc.Driver");
            System.out.println("Driver cargado exitosamente.");
            // Conectar a la base de datos
            return DriverManager.getConnection(URL, USER, PASSWORD);
        } catch (ClassNotFoundException e) {
            System.out.println("Error cargando el driver: " + e.getMessage());
            throw new SQLException("No se encontró el driver MySQL. Asegúrate de que mysql-connector-java.jar esté en el classpath.");
        }
    }
}

// Interfaz para operaciones CRUD
interface IDataPost<T> {
    void post(T entity) throws SQLException;
    T get(int id) throws SQLException;
    List<T> getAll() throws SQLException;
    void put(T entity) throws SQLException;
    void delete(int id) throws SQLException;
}

// Clase abstracta base para el controlador de datos
abstract class DataContext<T> implements IDataPost<T> {}

// Modelo para la entidad Customer
class Customer {
    private int customerId;
    private String firstName;
    private String lastName;
    private City city;

    public Customer(int customerId, String firstName, String lastName, City city) {
        this.customerId = customerId;
        this.firstName = firstName;
        this.lastName = lastName;
        this.city = city;
    }

    public int getCustomerId() { return customerId; }
    public void setCustomerId(int customerId) { this.customerId = customerId; }
    public String getFirstName() { return firstName; }
    public void setFirstName(String firstName) { this.firstName = firstName; }
    public String getLastName() { return lastName; }
    public void setLastName(String lastName) { this.lastName = lastName; }
    public City getCity() { return city; }
    public void setCity(City city) { this.city = city; }

    @Override
    public String toString() {
        return "Customer ID: " + customerId + ", Nombre: " + firstName + " " + lastName + ", Ciudad: " + city.getCityName();
    }
}

// Modelo para la entidad City
class City {
    private int cityId;
    private String cityName;
    private Country country;

    public City(int cityId, String cityName, Country country) {
        this.cityId = cityId;
        this.cityName = cityName;
        this.country = country;
    }

    public int getCityId() { return cityId; }
    public String getCityName() { return cityName; }
    public Country getCountry() { return country; }
}

// Modelo para la entidad Country
class Country {
    private int countryId;
    private String countryName;

    public Country(int countryId, String countryName) {
        this.countryId = countryId;
        this.countryName = countryName;
    }

    public int getCountryId() { return countryId; }
    public String getCountryName() { return countryName; }
}

// Controlador para manejar las operaciones de Customer en MySQL
class CustomerController extends DataContext<Customer> {
    public CustomerController() { super(); }

    @Override
    public void post(Customer customer) throws SQLException {
        String query = "INSERT INTO customer (first_name, last_name, city_id, store_id , address_id ) VALUES (?, ?, ?, ?,?)";
        try (Connection conn = DBConnection.getConnection();
             PreparedStatement stmt = conn.prepareStatement(query)) {
            stmt.setString(1, customer.getFirstName());
            stmt.setString(2, customer.getLastName());
            stmt.setInt(3, customer.getCity().getCityId());
            stmt.setInt(4, 1);
            stmt.setInt(5, 1);
            stmt.executeUpdate();
            System.out.println("Customer creado exitosamente.");
        }
    }

    @Override
    public Customer get(int id) throws SQLException {
        String query = "SELECT customer_id, first_name, last_name, city_id FROM customer WHERE customer_id = ?";
        try (Connection conn = DBConnection.getConnection();
             PreparedStatement stmt = conn.prepareStatement(query)) {
            stmt.setInt(1, id);
            ResultSet rs = stmt.executeQuery();
            if (rs.next()) {
                int customerId = rs.getInt("customer_id");
                String firstName = rs.getString("first_name");
                String lastName = rs.getString("last_name");
                City city = new City(rs.getInt("city_id"), "SampleCity", new Country(1, "SampleCountry"));
                return new Customer(customerId, firstName, lastName, city);
            }
        }
        return null;
    }

    @Override
    public List<Customer> getAll() throws SQLException {
        List<Customer> customers = new ArrayList<>();
        String query = "SELECT customer_id, first_name, last_name, city_id FROM customer";
        try (Connection conn = DBConnection.getConnection();
             PreparedStatement stmt = conn.prepareStatement(query);
             ResultSet rs = stmt.executeQuery()) {
            while (rs.next()) {
                int customerId = rs.getInt("customer_id");
                String firstName = rs.getString("first_name");
                String lastName = rs.getString("last_name");
                City city = new City(rs.getInt("city_id"), "SampleCity", new Country(1, "SampleCountry"));
                customers.add(new Customer(customerId, firstName, lastName, city));
            }
        }
        return customers;
    }


    @Override
    public void put(Customer customer) throws SQLException {
        String query = "UPDATE customer SET first_name = ?, last_name = ? WHERE customer_id = ?";
        try (Connection conn = DBConnection.getConnection();
             PreparedStatement stmt = conn.prepareStatement(query)) {
            stmt.setString(1, customer.getFirstName());
            stmt.setString(2, customer.getLastName());
            stmt.setInt(3, customer.getCustomerId());
            stmt.executeUpdate();
            System.out.println("Customer actualizado exitosamente.");
        }
    }

    @Override
    public void delete(int id) throws SQLException {
        String query = "DELETE FROM customer WHERE customer_id = ?";
        try (Connection conn = DBConnection.getConnection();
             PreparedStatement stmt = conn.prepareStatement(query)) {
            stmt.setInt(1, id);
            stmt.executeUpdate();
            System.out.println("Customer eliminado exitosamente.");
        }
    }
}

// Aplicación principal con interacción en consola
public class Main {
    private static final CustomerController customerController = new CustomerController();
    private static final Scanner scanner = new Scanner(System.in);

    public static void main(String[] args) {
        int option;
        do {
            System.out.println("1. Crear Customer");
            System.out.println("2. Buscar Customer");
            System.out.println("3. Actualizar Customer");
            System.out.println("4. Eliminar Customer");
            System.out.println("5. Salir");
            option = scanner.nextInt();
            scanner.nextLine();

            try {
                switch (option) {
                    case 1:
                        createCustomer();
                        break;
                    case 2:
                        searchCustomer();
                        break;
                    case 3:
                        updateCustomer();
                        break;
                    case 4:
                        deleteCustomer();
                        break;
                    case 5:
                        System.out.println("Saliendo...");
                        break;
                    default:
                        System.out.println("Opción no válida");
                        break;
                }
            } catch (SQLException e) {
                System.out.println("Error de base de datos: " + e.getMessage());
            }
        } while (option != 5);
    }

    private static void createCustomer() throws SQLException {
        System.out.print("Nombre: ");
        String firstName = scanner.nextLine();
        System.out.print("Apellido: ");
        String lastName = scanner.nextLine();
        Customer customer = new Customer(0, firstName, lastName, new City(1, "SampleCity", new Country(1, "SampleCountry")));
        customerController.post(customer);
    }

    private static void searchCustomer() throws SQLException {
        System.out.print("ID de Customer: ");
        int id = scanner.nextInt();
        scanner.nextLine();
        Customer customer = customerController.get(id);
        System.out.println(customer != null ? customer : "Customer no encontrado.");
    }

    private static void updateCustomer() throws SQLException {
        System.out.print("ID de Customer a actualizar: ");
        int id = scanner.nextInt();
        scanner.nextLine();
        Customer customer = customerController.get(id);
        if (customer != null) {
            System.out.print("Nuevo Nombre: ");
            customer.setFirstName(scanner.nextLine());
            System.out.print("Nuevo Apellido: ");
            customer.setLastName(scanner.nextLine());
            customerController.put(customer);
        } else {
            System.out.println("Customer no encontrado.");
        }
    }

    private static void deleteCustomer() throws SQLException {
        System.out.print("ID de Customer a eliminar: ");
        int id = scanner.nextInt();
        scanner.nextLine();
        customerController.delete(id);
    }
}
