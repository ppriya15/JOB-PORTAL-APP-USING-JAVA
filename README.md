# JOB-PORTAL-APP-USING-JAVA
import java.awt.*;
import java.lang.*;
import java.security.MessageDigest;
import java.security.NoSuchAlgorithmException;
import java.util.ArrayList;
import java.util.HashMap;
import java.util.List;
import java.util.Map;
import javax.swing.*;
// Job class to represent job details
class Job {
    private String title;
    private String description;
    private String company;
    private double salary;
    private String postedBy;

    public Job(String title, String description, String company, double salary, String postedBy) {
        this.title = title;
        this.description = description;
        this.company = company;
        this.salary = salary;
        this.postedBy = postedBy;
    }

    public String getTitle() {
        return title;
    }

    @Override
    public String toString() {
        return "<html><b>Title:</b> " + title + "<br><b>Company:</b> " + company +
                "<br><b>Salary:</b> $" + salary + "<br><b>Description:</b> " + description +
                "<br><b>Posted By:</b> " + postedBy + "</html>";
    }
}

// Main application
public class JobPortalApp extends JFrame {
    private final Map<String, String> users = new HashMap<>(); // Stores username-password pairs
    private final Map<String, String> roles = new HashMap<>(); // Stores username-role pairs
    private final List<Job> jobList = new ArrayList<>(); // Stores job posts
    private final Map<String, List<Job>> appliedJobs = new HashMap<>(); // Tracks applied jobs
    private String currentUser = null;

    public JobPortalApp() {
        setTitle("Online Job Portal");
        setSize(600, 400);
        setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
        setLocationRelativeTo(null);
        setLayout(new CardLayout());
        showMainScreen();
    }

    private void showMainScreen() {
        JPanel mainPanel = new JPanel(new GridLayout(3, 1, 10, 10));
        mainPanel.setBorder(BorderFactory.createEmptyBorder(20, 20, 20, 20));

        JLabel welcomeLabel = new JLabel("Welcome to the Online Job Portal", SwingConstants.CENTER);
        welcomeLabel.setFont(new Font("Arial", Font.BOLD, 18));
        mainPanel.add(welcomeLabel);

        JButton registerButton = new JButton("Register");
        JButton loginButton = new JButton("Login");

        mainPanel.add(registerButton);
        mainPanel.add(loginButton);

        setContentPane(mainPanel);

        registerButton.addActionListener(e -> showRegisterScreen());
        loginButton.addActionListener(e -> showLoginScreen());

        setVisible(true);
    }

    private void showRegisterScreen() {
        JPanel registerPanel = new JPanel(new GridLayout(5, 2, 10, 10));
        registerPanel.setBorder(BorderFactory.createEmptyBorder(20, 20, 20, 20));

        JLabel registerLabel = new JLabel("Register", SwingConstants.CENTER);
        registerLabel.setFont(new Font("Arial", Font.BOLD, 18));
        registerPanel.add(registerLabel);
        registerPanel.add(new JLabel()); // For alignment

        JTextField usernameField = new JTextField();
        JPasswordField passwordField = new JPasswordField();
        JComboBox<String> roleComboBox = new JComboBox<>(new String[]{"Employer", "Job Seeker"});

        registerPanel.add(new JLabel("Username:"));
        registerPanel.add(usernameField);
        registerPanel.add(new JLabel("Password:"));
        registerPanel.add(passwordField);
        registerPanel.add(new JLabel("Role:"));
        registerPanel.add(roleComboBox);

        JButton registerButton = new JButton("Register");
        JButton backButton = new JButton("Back");

        registerPanel.add(registerButton);
        registerPanel.add(backButton);

        setContentPane(registerPanel);

        registerButton.addActionListener(e -> {
            String username = usernameField.getText().trim();
            String password = new String(passwordField.getPassword()).trim();
            String role = (String) roleComboBox.getSelectedItem();

            if (username.isEmpty() || password.isEmpty()) {
                JOptionPane.showMessageDialog(this, "All fields are required!", "Error", JOptionPane.ERROR_MESSAGE);
            } else if (users.containsKey(username)) {
                JOptionPane.showMessageDialog(this, "Username already exists!", "Error", JOptionPane.ERROR_MESSAGE);
            } else {
                users.put(username, hashPassword(password));
                roles.put(username, role);
                if ("Job Seeker".equals(role)) {
                    appliedJobs.put(username, new ArrayList<>());
                }
                JOptionPane.showMessageDialog(this, "Registration successful!", "Success", JOptionPane.INFORMATION_MESSAGE);
                showMainScreen();
            }
        });

        backButton.addActionListener(e -> showMainScreen());
        setVisible(true);
    }

    private void showLoginScreen() {
        JPanel loginPanel = new JPanel(new GridLayout(4, 2, 10, 10));
        loginPanel.setBorder(BorderFactory.createEmptyBorder(20, 20, 20, 20));

        JLabel loginLabel = new JLabel("Login", SwingConstants.CENTER);
        loginLabel.setFont(new Font("Arial", Font.BOLD, 18));
        loginPanel.add(loginLabel);
        loginPanel.add(new JLabel()); // For alignment

        JTextField usernameField = new JTextField();
        JPasswordField passwordField = new JPasswordField();

        loginPanel.add(new JLabel("Username:"));
        loginPanel.add(usernameField);
        loginPanel.add(new JLabel("Password:"));
        loginPanel.add(passwordField);

        JButton loginButton = new JButton("Login");
        JButton backButton = new JButton("Back");

        loginPanel.add(loginButton);
        loginPanel.add(backButton);

        setContentPane(loginPanel);

        loginButton.addActionListener(e -> {
            String username = usernameField.getText().trim();
            String password = new String(passwordField.getPassword()).trim();

            if (users.containsKey(username) && users.get(username).equals(hashPassword(password))) {
                currentUser = username;
                String role = roles.get(username);
                if ("Employer".equals(role)) {
                    showEmployerPanel();
                } else {
                    showJobSeekerPanel();
                }
            } else {
                JOptionPane.showMessageDialog(this, "Invalid username or password!", "Error", JOptionPane.ERROR_MESSAGE);
            }
        });

        backButton.addActionListener(e -> showMainScreen());
        setVisible(true);
    }

    private void showEmployerPanel() {
        JFrame employerFrame = new JFrame("Employer Panel");
        employerFrame.setSize(500, 400);
        employerFrame.setLocationRelativeTo(null);
        employerFrame.setLayout(new BorderLayout());

        JLabel titleLabel = new JLabel("Post a Job", SwingConstants.CENTER);
        titleLabel.setFont(new Font("Arial", Font.BOLD, 16));
        employerFrame.add(titleLabel, BorderLayout.NORTH);

        JPanel formPanel = new JPanel(new GridLayout(5, 2, 10, 10));
        formPanel.setBorder(BorderFactory.createEmptyBorder(10, 10, 10, 10));

        JTextField titleField = new JTextField();
        JTextField companyField = new JTextField();
        JTextField salaryField = new JTextField();
        JTextArea descriptionArea = new JTextArea(3, 20);

        formPanel.add(new JLabel("Job Title:"));
        formPanel.add(titleField);
        formPanel.add(new JLabel("Company Name:"));
        formPanel.add(companyField);
        formPanel.add(new JLabel("Salary:"));
        formPanel.add(salaryField);
        formPanel.add(new JLabel("Description:"));
        formPanel.add(new JScrollPane(descriptionArea));

        employerFrame.add(formPanel, BorderLayout.CENTER);

        JButton postJobButton = new JButton("Post Job");
        JButton logoutButton = new JButton("Logout");

        JPanel buttonPanel = new JPanel();
        buttonPanel.add(postJobButton);
        buttonPanel.add(logoutButton);

        employerFrame.add(buttonPanel, BorderLayout.SOUTH);

        postJobButton.addActionListener(e -> {
            String title = titleField.getText().trim();
            String company = companyField.getText().trim();
            double salary;
            try {
                salary = Double.parseDouble(salaryField.getText().trim());
            } catch (NumberFormatException ex) {
                JOptionPane.showMessageDialog(employerFrame, "Invalid salary input!", "Error", JOptionPane.ERROR_MESSAGE);
                return;
            }
            String description = descriptionArea.getText().trim();

            if (title.isEmpty() || company.isEmpty() || description.isEmpty()) {
                JOptionPane.showMessageDialog(employerFrame, "All fields are required!", "Error", JOptionPane.ERROR_MESSAGE);
                return;
            }

            jobList.add(new Job(title, description, company, salary, currentUser));
            JOptionPane.showMessageDialog(employerFrame, "Job posted successfully!", "Success", JOptionPane.INFORMATION_MESSAGE);
        });

        logoutButton.addActionListener(e -> {
            employerFrame.dispose();
            currentUser = null;
            showMainScreen();
        });

        employerFrame.setVisible(true);
    }

    private void showJobSeekerPanel() {
        JFrame seekerFrame = new JFrame("Job Seeker Panel");
        seekerFrame.setSize(500, 400);
        seekerFrame.setLocationRelativeTo(null);
        seekerFrame.setLayout(new BorderLayout());

        JLabel titleLabel = new JLabel("Available Jobs", SwingConstants.CENTER);
        titleLabel.setFont(new Font("Arial", Font.BOLD, 16));
        seekerFrame.add(titleLabel, BorderLayout.NORTH);

        JPanel jobPanel = new JPanel();
        jobPanel.setLayout(new BoxLayout(jobPanel, BoxLayout.Y_AXIS));
        JScrollPane scrollPane = new JScrollPane(jobPanel);
        seekerFrame.add(scrollPane, BorderLayout.CENTER);

        for (Job job : jobList) {
            JPanel jobDetailsPanel = new JPanel(new BorderLayout());
            jobDetailsPanel.setBorder(BorderFactory.createTitledBorder(job.getTitle()));

            JLabel jobDetails = new JLabel(job.toString());
            jobDetailsPanel.add(jobDetails, BorderLayout.CENTER);

            JButton applyButton = new JButton("Apply");
            jobDetailsPanel.add(applyButton, BorderLayout.SOUTH);

            applyButton.addActionListener(e -> {
                appliedJobs.get(currentUser).add(job);
                JOptionPane.showMessageDialog(seekerFrame, "You applied for " + job.getTitle(), "Success", JOptionPane.INFORMATION_MESSAGE);
            });

            jobPanel.add(jobDetailsPanel);
        }

        JButton logoutButton = new JButton("Logout");
        seekerFrame.add(logoutButton, BorderLayout.SOUTH);

        logoutButton.addActionListener(e -> {
            seekerFrame.dispose();
            currentUser = null;
            showMainScreen();
        });

        seekerFrame.setVisible(true);
    }

    private String hashPassword(String password) {
        try {
            MessageDigest md = MessageDigest.getInstance("SHA-256");
            byte[] hash = md.digest(password.getBytes());
            StringBuilder hexString = new StringBuilder();
            for (byte b : hash) {
                String hex = Integer.toHexString(0xff & b);
                if (hex.length() == 1) hexString.append('0');
                hexString.append(hex);
            }
            return hexString.toString();
        } catch (NoSuchAlgorithmException e) {
            throw new RuntimeException("Error hashing password", e);
        }
    }

    public static void main(String[] args) {
        SwingUtilities.invokeLater(JobPortalApp::new);
    }
}
