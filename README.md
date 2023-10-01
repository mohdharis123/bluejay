# bluejay
import org.apache.poi.ss.usermodel.*;
import org.apache.poi.xssf.usermodel.XSSFWorkbook;
import org.eclipse.jgit.api.Git;
import org.eclipse.jgit.api.errors.GitAPIException;
import org.eclipse.jgit.internal.storage.file.FileRepository;
import org.eclipse.jgit.lib.Repository;

import java.io.File;
import java.io.FileInputStream;
import java.io.IOException;
import java.text.SimpleDateFormat;
import java.util.ArrayList;
import java.util.Date;
import java.util.List;

public class Main {

    public static void main(String[] args) {
        String excelFilePath = "Assignment_Timecard.xlsx";

        try {
            List<Employee> employees = readEmployeesFromExcel(excelFilePath);

            String gitRepositoryPath = "your_git_repository_path";
            createAndPushToGit(employees, gitRepositoryPath);

        } catch (IOException | GitAPIException e) {
            e.printStackTrace();
        }
    }

    private static List<Employee> readEmployeesFromExcel(String filePath) throws IOException {
        List<Employee> employees = new ArrayList<>();

        FileInputStream inputStream = new FileInputStream(new File(filePath));
        Workbook workbook = new XSSFWorkbook(inputStream);
        Sheet sheet = workbook.getSheetAt(0);

        for (Row row : sheet) {
            if (row.getRowNum() == 0) {
                continue;
            }

            String name = row.getCell(0).getStringCellValue();
            String position = row.getCell(1).getStringCellValue();
            String workDates = row.getCell(2).getStringCellValue();
            double shiftHours = row.getCell(3).getNumericCellValue();

            employees.add(new Employee(name, position, workDates, shiftHours));
        }

        workbook.close();
        inputStream.close();

        return employees;
    }

    private static void createAndPushToGit(List<Employee> employees, String repositoryPath) throws GitAPIException {
        Repository existingRepo = new FileRepository(repositoryPath + "/.git");
        Git git = new Git(existingRepo);

        git.branchCreate().setName("employee-analysis").call();
        git.checkout().setName("employee-analysis").call();

        for (Employee employee : employees) {
            if (employee.hasConsecutiveDays()) {
                System.out.println(employee.getName() + " (" + employee.getPosition() + ") has worked for 7 consecutive days.");
            }

            if (employee.hasShortTimeBetweenShifts()) {
                System.out.println(employee.getName() + " (" + employee.getPosition() + ") has less than 10 hours between shifts but greater than 1 hour.");
            }

            if (employee.hasLongShift()) {
                System.out.println(employee.getName() + " (" + employee.getPosition() + ") has worked for more than 14 hours in a single shift.");
            }
        }

        git.add().addFilepattern(".").call();
        git.commit().setMessage("Employee analysis results").call();
        git.push().call();
    }
}

class Employee {
    private String name;
    private String position;
    private String workDates;
    private double shiftHours;

    public Employee(String name, String position, String workDates, double shiftHours) {
        this.name = name;
        this.position = position;
        this.workDates = workDates;
        this.shiftHours = shiftHours;
    }

    public String getName() {
        return name;
    }

    public String getPosition() {
        return position;
    }

    public boolean hasConsecutiveDays() {
        return false; // Placeholder
    }

    public boolean hasShortTimeBetweenShifts() {
        return false; // Placeholder
    }

    public boolean hasLongShift() {
        return false; // Placeholder
    }
}
