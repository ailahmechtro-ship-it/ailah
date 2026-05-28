
/*
 * ============================================================
 *   STUDENT FEE RECORD SYSTEM
 *   ESP32-PC Hybrid Database Management System
 *   Course Project — CEA
 * ============================================================
 *
 *  Features:
 *   - Login/Password authentication (admin only)
 *   - CSV file linked as the live database (Excel-compatible)
 *   - initializeDatabase()  — creates CSV with headers if missing
 *   - isUnique(id)          — prevents duplicate Student IDs
 *   - appendRecord(data)    — adds a new fee record
/*
============================================================
            STUDENT FEE TRACKER SYSTEM
============================================================

Project Features:
1. Admin Login System
2. Add Student Fee Record
3. Search Student Record
4. Update Student Record
5. Delete Student Record
6. Display All Student Records
7. Generate Fee Due Report
8. Automatic CSV File Handling
9. Excel Compatible Database

Database File:
student_fees.csv

This CSV file can be opened directly in Microsoft Excel.

============================================================
HOW TO COMPILE:
g++ -std=c++17 -o fee_tracker fee_tracker.cpp

HOW TO RUN:
./fee_tracker
============================================================
*/

#include <iostream>
#include <fstream>
#include <sstream>
#include <vector>
#include <string>
#include <iomanip>
#include <algorithm>

using namespace std;

/*============================================================
                    GLOBAL CONSTANTS
============================================================*/

// CSV file name
const string FILE_NAME = "student_fees.csv";

// Login credentials
const string ADMIN_USERNAME = "admin";
const string ADMIN_PASSWORD = "1234";

// Total number of columns in CSV file
const int TOTAL_COLUMNS = 8;

/*============================================================
                FUNCTION DECLARATIONS
============================================================*/

void initializeDatabase();
bool login();

void addStudent();
void displayAllStudents();
void searchStudent();
void updateStudent();
void deleteStudent();
void generateReport();

bool isUniqueID(string id);
string searchByID(string id);

vector<string> splitCSV(string line);
void appendRecord(string data);
bool updateRecord(string id, string newData);

void mainMenu();

/*============================================================
                SPLIT CSV FUNCTION
============================================================

This function separates CSV data into tokens.

Example:
Input:
S101,Ali,CS,2,50000,25000,2026-06-01,Partial

Output Vector:
[0] S101
[1] Ali
[2] CS
etc.

============================================================*/

vector<string> splitCSV(string line)
{
    vector<string> tokens;
    string token;
    stringstream ss(line);

    while (getline(ss, token, ','))
    {
        tokens.push_back(token);
    }

    return tokens;
}

/*============================================================
            DATABASE INITIALIZATION FUNCTION
============================================================

If CSV file does not exist:
1. Create file
2. Add headings

============================================================*/

void initializeDatabase()
{
    ifstream checkFile(FILE_NAME);

    // If file already exists
    if (checkFile.good())
    {
        cout << "\nDatabase Found Successfully.\n";
        checkFile.close();
        return;
    }

    checkFile.close();

    // Create new CSV file
    ofstream file(FILE_NAME);

    // Add column headings
    file << "StudentID,Name,Department,Semester,TotalFee,PaidFee,DueDate,Status\n";

    file.close();

    cout << "\nNew Database Created Successfully.\n";
}

/*============================================================
                LOGIN FUNCTION
============================================================

Allows only admin access.

3 login attempts allowed.

============================================================*/

bool login()
{
    string username, password;

    for (int i = 1; i <= 3; i++)
    {
        cout << "\n========== LOGIN ==========\n";

        cout << "Enter Username: ";
        getline(cin, username);

        cout << "Enter Password: ";
        getline(cin, password);

        // Check credentials
        if (username == ADMIN_USERNAME &&
            password == ADMIN_PASSWORD)
        {
            cout << "\nLogin Successful!\n";
            return true;
        }

        cout << "\nIncorrect Username or Password.\n";
        cout << "Attempts Left: " << 3 - i << endl;
    }

    cout << "\nToo Many Failed Attempts.\n";
    return false;
}

/*============================================================
            CHECK UNIQUE STUDENT ID
============================================================

Prevents duplicate Student IDs.

Returns:
true  -> ID is unique
false -> ID already exists

============================================================*/

bool isUniqueID(string id)
{
    ifstream file(FILE_NAME);

    string line;

    // Skip heading row
    getline(file, line);

    while (getline(file, line))
    {
        vector<string> row = splitCSV(line);

        if (row[0] == id)
        {
            file.close();
            return false;
        }
    }

    file.close();
    return true;
}

/*============================================================
                APPEND RECORD FUNCTION
============================================================

Adds a new record at end of CSV file.

============================================================*/

void appendRecord(string data)
{
    ofstream file(FILE_NAME, ios::app);

    file << data << endl;

    file.close();
}

/*============================================================
                ADD STUDENT FUNCTION
============================================================*/

void addStudent()
{
    string id, name, department;
    string semester, totalFee;
    string paidFee, dueDate;
    string status;

    cout << "\n========== ADD STUDENT ==========\n";

    cout << "Enter Student ID: ";
    getline(cin, id);

    // Check duplicate ID
    if (!isUniqueID(id))
    {
        cout << "\nERROR: Student ID already exists.\n";
        return;
    }

    cout << "Enter Student Name: ";
    getline(cin, name);

    cout << "Enter Department: ";
    getline(cin, department);

    cout << "Enter Semester: ";
    getline(cin, semester);

    cout << "Enter Total Fee: ";
    getline(cin, totalFee);

    cout << "Enter Paid Fee: ";
    getline(cin, paidFee);

    cout << "Enter Due Date (YYYY-MM-DD): ";
    getline(cin, dueDate);

    // Calculate fee status
    double total = stod(totalFee);
    double paid = stod(paidFee);

    if (paid == 0)
    {
        status = "Unpaid";
    }
    else if (paid >= total)
    {
        status = "Paid";
    }
    else
    {
        status = "Partial";
    }

    // Prepare CSV row
    string record =
        id + "," +
        name + "," +
        department + "," +
        semester + "," +
        totalFee + "," +
        paidFee + "," +
        dueDate + "," +
        status;

    // Save record
    appendRecord(record);

    cout << "\nStudent Record Added Successfully.\n";
    cout << "Data Saved in Excel CSV File.\n";
}

/*============================================================
                DISPLAY ALL RECORDS
============================================================*/

void displayAllStudents()
{
    ifstream file(FILE_NAME);

    string line;

    cout << "\n========== ALL STUDENT RECORDS ==========\n\n";

    // Print headings
    cout << left
         << setw(12) << "ID"
         << setw(20) << "Name"
         << setw(15) << "Department"
         << setw(10) << "Semester"
         << setw(12) << "TotalFee"
         << setw(12) << "PaidFee"
         << setw(15) << "DueDate"
         << setw(10) << "Status"
         << endl;

    cout << string(105, '-') << endl;

    // Skip header row
    getline(file, line);

    // Display all rows
    while (getline(file, line))
    {
        vector<string> row = splitCSV(line);

        if (row.size() >= TOTAL_COLUMNS)
        {
            cout << left
                 << setw(12) << row[0]
                 << setw(20) << row[1]
                 << setw(15) << row[2]
                 << setw(10) << row[3]
                 << setw(12) << row[4]
                 << setw(12) << row[5]
                 << setw(15) << row[6]
                 << setw(10) << row[7]
                 << endl;
        }
    }

    file.close();
}

/*============================================================
                SEARCH BY ID FUNCTION
============================================================*/

string searchByID(string id)
{
    ifstream file(FILE_NAME);

    string line;

    // Skip headings
    getline(file, line);

    while (getline(file, line))
    {
        vector<string> row = splitCSV(line);

        if (row[0] == id)
        {
            file.close();
            return line;
        }
    }

    file.close();

    return "";
}

/*============================================================
                SEARCH STUDENT FUNCTION
============================================================*/

void searchStudent()
{
    string id;

    cout << "\nEnter Student ID to Search: ";
    getline(cin, id);

    string result = searchByID(id);

    if (result == "")
    {
        cout << "\nStudent Record Not Found.\n";
        return;
    }

    vector<string> row = splitCSV(result);

    cout << "\n========== STUDENT RECORD ==========\n";

    cout << "Student ID : " << row[0] << endl;
    cout << "Name       : " << row[1] << endl;
    cout << "Department : " << row[2] << endl;
    cout << "Semester   : " << row[3] << endl;
    cout << "Total Fee  : " << row[4] << endl;
    cout << "Paid Fee   : " << row[5] << endl;
    cout << "Due Date   : " << row[6] << endl;
    cout << "Status     : " << row[7] << endl;
}

/*============================================================
                UPDATE RECORD FUNCTION
============================================================

Uses temporary file method.

1. Read old file
2. Write updated data into temp file
3. Replace old file

============================================================*/

bool updateRecord(string id, string newData)
{
    ifstream inputFile(FILE_NAME);

    ofstream tempFile("temp.csv");

    string line;

    bool found = false;

    // Copy header row
    getline(inputFile, line);
    tempFile << line << endl;

    while (getline(inputFile, line))
    {
        vector<string> row = splitCSV(line);

        if (row[0] == id)
        {
            found = true;

            // Replace old record
            if (newData != "")
            {
                tempFile << newData << endl;
            }
        }
        else
        {
            tempFile << line << endl;
        }
    }

    inputFile.close();
    tempFile.close();

    remove(FILE_NAME.c_str());
    rename("temp.csv", FILE_NAME.c_str());

    return found;
}

/*============================================================
                UPDATE STUDENT FUNCTION
============================================================*/

void updateStudent()
{
    string id;

    cout << "\nEnter Student ID to Update: ";
    getline(cin, id);

    string existing = searchByID(id);

    if (existing == "")
    {
        cout << "\nRecord Not Found.\n";
        return;
    }

    vector<string> row = splitCSV(existing);

    string name, department;
    string semester, totalFee;
    string paidFee, dueDate;
    string status;

    cout << "\nEnter New Name: ";
    getline(cin, name);

    cout << "Enter New Department: ";
    getline(cin, department);

    cout << "Enter New Semester: ";
    getline(cin, semester);

    cout << "Enter New Total Fee: ";
    getline(cin, totalFee);

    cout << "Enter New Paid Fee: ";
    getline(cin, paidFee);

    cout << "Enter New Due Date: ";
    getline(cin, dueDate);

    // Recalculate status
    double total = stod(totalFee);
    double paid = stod(paidFee);

    if (paid == 0)
    {
        status = "Unpaid";
    }
    else if (paid >= total)
    {
        status = "Paid";
    }
    else
    {
        status = "Partial";
    }

    // Create updated record
    string updatedRecord =
        id + "," +
        name + "," +
        department + "," +
        semester + "," +
        totalFee + "," +
        paidFee + "," +
        dueDate + "," +
        status;

    updateRecord(id, updatedRecord);

    cout << "\nRecord Updated Successfully.\n";
}

/*============================================================
                DELETE STUDENT FUNCTION
============================================================*/

void deleteStudent()
{
    string id;

    cout << "\nEnter Student ID to Delete: ";
    getline(cin, id);

    if (updateRecord(id, ""))
    {
        cout << "\nRecord Deleted Successfully.\n";
    }
    else
    {
        cout << "\nRecord Not Found.\n";
    }
}

/*============================================================
                GENERATE REPORT FUNCTION
============================================================

Displays:
1. Unpaid Students
2. Partial Paid Students
3. Total Due Amount

============================================================*/

void generateReport()
{
    ifstream file(FILE_NAME);

    string line;

    double totalDue = 0;

    cout << "\n========== FEE DUE REPORT ==========\n";

    // Skip header row
    getline(file, line);

    while (getline(file, line))
    {
        vector<string> row = splitCSV(line);

        if (row.size() >= TOTAL_COLUMNS)
        {
            double totalFee = stod(row[4]);
            double paidFee = stod(row[5]);

            double due = totalFee - paidFee;

            if (due > 0)
            {
                cout << "\nStudent ID : " << row[0];
                cout << "\nName       : " << row[1];
                cout << "\nDue Amount : " << due << endl;

                totalDue += due;
            }
        }
    }

    cout << "\n----------------------------------\n";
    cout << "Total Outstanding Fee: " << totalDue << endl;

    file.close();
}

/*============================================================
                    MAIN MENU FUNCTION
============================================================*/

void mainMenu()
{
    int choice;

    do
    {
        cout << "\n========== MAIN MENU ==========\n";

        cout << "1. Add Student Record\n";
        cout << "2. Search Student Record\n";
        cout << "3. Update Student Record\n";
        cout << "4. Delete Student Record\n";
        cout << "5. Display All Records\n";
        cout << "6. Generate Fee Report\n";
        cout << "0. Exit Program\n";

        cout << "\nEnter Choice: ";
        cin >> choice;

        // Clear input buffer
        cin.ignore();

        switch (choice)
        {
            case 1:
                addStudent();
                break;

            case 2:
                searchStudent();
                break;

            case 3:
                updateStudent();
                break;

            case 4:
                deleteStudent();
                break;

            case 5:
                displayAllStudents();
                break;

            case 6:
                generateReport();
                break;

            case 0:
                cout << "\nExiting Program...\n";
                break;

            default:
                cout << "\nInvalid Choice.\n";
        }

    } while (choice != 0);
}

/*============================================================
                        MAIN FUNCTION
============================================================*/

int main()
{
    // Create database if missing
    initializeDatabase();

    // Login check
    if (!login())
    {
        return 0;
    }

    // Start menu system
    mainMenu();

    return 0;
}

/*
============================================================
                EXCEL FILE INTEGRATION
============================================================

The CSV file is updated automatically whenever:
1. A student is added
2. A student is updated
3. A student is deleted

HOW TO OPEN IN EXCEL:
1. Open Microsoft Excel
2. Click File -> Open
3. Select student_fees.csv

HOW TO REFRESH:
Press CTRL + F5 in Excel after making changes.

============================================================
*/
