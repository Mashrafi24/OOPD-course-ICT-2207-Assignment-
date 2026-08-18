### 📘 L10 — Spring Boot REST API with JPA/ORM
# 10.1 — Setting up Spring Boot + REST + JPA

Create a Spring Boot project (via Spring Initializr) with dependencies: spring-boot-starter-web (for REST), spring-boot-starter-data-jpa (for ORM), and the MySQL connector.
Configure the database connection in application.properties (URL, username, password).
Write an @Entity class (Model), a Repository interface, and a @RestController (endpoints). Role of embedded Tomcat: Spring Boot bundles a Tomcat server inside the app itself, so you don't need to install or configure a separate server — running the app starts a working web server automatically.


10.2 — Practical: JPA Entity + Repository

import jakarta.persistence.*;

@Entity
public class Student {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    private String name;
    private double cgpa;

    // getters and setters
    public Long getId() { return id; }
    public String getName() { return name; }
    public double getCgpa() { return cgpa; }
    public void setName(String name) { this.name = name; }
    public void setCgpa(double cgpa) { this.cgpa = cgpa; }
}
import org.springframework.data.jpa.repository.JpaRepository;

public interface StudentRepository extends JpaRepository<Student, Long> {
    // no code needed — JpaRepository already gives save(), findAll(), findById(), etc.
}
10.3 — Practical: REST Controller

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.*;
import java.util.List;

@RestController
@RequestMapping("/students")
public class StudentController {

    @Autowired
    private StudentRepository repo;

    @GetMapping
    public List<Student> getAll() {
        return repo.findAll();
    }

    @PostMapping
    public Student add(@RequestBody Student student) {
        return repo.save(student);
    }
}
### 📘 L11 — Servlet CRUD — District Quiz Game
# 11.1 — DB schema design A quiz needs to track categories, questions, players, and scores:

CATEGORY(category_id PK, category_name) — e.g. Crops, Geography, Institutions.
QUESTION(question_id PK, category_id FK, question_text, option_a, option_b, option_c, option_d, correct_option) — each question belongs to one category.
PLAYER(player_id PK, name) — the person playing.
PLAYER_SCORE(score_id PK, player_id FK, total_score, played_on) — one row per game session, linking back to the player. Justification: Separating QUESTION from CATEGORY lets you filter/add questions per topic easily. Separating PLAYER from PLAYER_SCORE lets one player have many play sessions (score history) instead of just one score.


# 11.2 — Practical: Save player score

import java.sql.*;
import jakarta.servlet.*;
import jakarta.servlet.http.*;

public class SaveScoreServlet extends HttpServlet {
    protected void doPost(HttpServletRequest request, HttpServletResponse response)
            throws ServletException, java.io.IOException {
        String name = request.getParameter("name");
        int score = Integer.parseInt(request.getParameter("score"));

        String url = "jdbc:mysql://localhost:3306/quiz_db";
        try (Connection con = DriverManager.getConnection(url, "root", "password");
             PreparedStatement ps = con.prepareStatement(
                 "INSERT INTO PLAYER_SCORE (player_id, total_score, played_on) " +
                 "VALUES ((SELECT player_id FROM PLAYER WHERE name = ?), ?, NOW())")) {
            ps.setString(1, name);
            ps.setInt(2, score);
            ps.executeUpdate();
            response.getWriter().println("Score saved for " + name);
        } catch (SQLException e) {
            response.getWriter().println("Error: " + e.getMessage());
        }
    }
}

# 11.3 — Practical: Quiz logic

import java.util.Scanner;

class Question {
    String text, optA, optB, optC, optD, correct;
    Question(String text, String a, String b, String c, String d, String correct) {
        this.text = text; this.optA = a; this.optB = b;
        this.optC = c; this.optD = d; this.correct = correct;
    }
}

public class QuizGame {
    public static void main(String[] args) {
        Question[] questions = {
            new Question("Main crop of the district?", "A) Rice", "B) Wheat", "C) Tea", "D) Cotton", "A"),
            new Question("Which river flows through the district?", "A) Padma", "B) Meghna", "C) Jamuna", "D) Surma", "B"),
            new Question("Oldest academic institution?", "A) High School", "B) College", "C) University", "D) Madrasa", "C")
        };

        Scanner sc = new Scanner(System.in);
        int score = 0;

        for (Question q : questions) {
            System.out.println(q.text);
            System.out.println(q.optA + "  " + q.optB + "  " + q.optC + "  " + q.optD);
            System.out.print("Your answer (A/B/C/D): ");
            String ans = sc.next().toUpperCase();
            if (ans.equals(q.correct)) {
                score++;
                System.out.println("Correct!\n");
            } else {
                System.out.println("Wrong! Correct answer: " + q.correct + "\n");
            }
        }

        System.out.println("Final Score: " + score + "/" + questions.length);
    }
}


#### 📘 L12 — GoF Design Patterns

# 12.1 — Creational patterns (5)

Pattern	Purpose
Singleton	Ensure only one instance of a class exists, with one global access point
Factory Method	Let subclasses decide which exact object to create
Abstract Factory	Create families of related objects without specifying their exact classes
Builder	Build a complex object step-by-step, separately from its final representation
Prototype	Create new objects by cloning an existing one


# 12.2 — Structural patterns (7)

Pattern	Purpose
Adapter	Convert one interface into another one the client expects
Bridge	Separate an abstraction from its implementation so both can change independently
Composite	Treat single objects and groups of objects the same way (tree structures)
Decorator	Add new behavior to an object dynamically, without changing its class
Facade	Give a simple front-door interface to a complicated subsystem
Flyweight	Share common data between many small objects to save memory
Proxy	Provide a stand-in object that controls access to the real object

# 12.3 — Practical: Singleton (thread-safe) + Adapter

// Thread-safe Singleton
class Singleton {
    private static volatile Singleton instance;

    private Singleton() {}

    public static Singleton getInstance() {
        if (instance == null) {
            synchronized (Singleton.class) {
                if (instance == null) {
                    instance = new Singleton();
                }
            }
        }
        return instance;
    }
}

// Adapter pattern
interface Target {
    void request();
}

class Adaptee {
    public void specificRequest() {
        System.out.println("Adaptee's specific behavior");
    }
}

class Adapter implements Target {
    private Adaptee adaptee;

    public Adapter(Adaptee adaptee) {
        this.adaptee = adaptee;
    }

    @Override
    public void request() {
        adaptee.specificRequest(); // delegates the call
    }
}

public class Main {
    public static void main(String[] args) {
        Singleton s1 = Singleton.getInstance();
        Singleton s2 = Singleton.getInstance();
        System.out.println("Same instance? " + (s1 == s2)); // true

        Target t = new Adapter(new Adaptee());
        t.request(); // "Adaptee's specific behavior"
    }
}
