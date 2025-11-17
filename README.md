# studentd_deta_bas
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

#define DATA_FILE "students.dat"
#define NAME_LEN 50

typedef struct
{
    int id;
    char name[NAME_LEN];
    int age;
    float marks;
} Student;

void read_line(char *buffer, int size)
{
    if (fgets(buffer, size, stdin) != NULL)
    {
        size_t ln = strlen(buffer);
        if (ln > 0 && buffer[ln - 1] == '\n')
            buffer[ln - 1] = '\0';
    }
}
void add_student()
{
    Student s;
    FILE *fp = fopen(DATA_FILE, "ab");
    if (!fp)
    {
        perror("Unable to open data file for appending");
        return;
    }
    printf("Enter Student ID: ");
    if (scanf("%d", &s.id) != 1)
    {
        while (getchar() != '\n')
            ;
        printf("Invalid input.\n");
        fclose(fp);
        return;
    }
    while (getchar() != '\n');
    printf("Enter Name: ");
    read_line(s.name, NAME_LEN);
    printf("Enter Age: ");
    if (scanf("%d", &s.age) != 1)
    {
        while (getchar() != '\n')
            ;
        printf("Invalid input.\n");
        fclose(fp);
        return;
    }
    while (getchar() != '\n');
    printf("Enter Marks (float): ");
    if (scanf("%f", &s.marks) != 1)
    {
        while (getchar() != '\n')
            ;
        printf("Invalid input.\n");
        fclose(fp);
        return;
    }
    while (getchar() != '\n');
    if (fwrite(&s, sizeof(Student), 1, fp) != 1)
    {
        perror("Write error");
    }
    else
    {
        printf("Student added successfully.\n");
    }

    fclose(fp);
}
void list_students()
{
    FILE *fp = fopen(DATA_FILE, "rb");
    if (!fp)
    {
        printf("No records found.\n");
        return;
    }

    Student s;
    printf("\n--- Students ---\n");
    printf("%-6s %-20s %-6s %-6s\n", "ID", "Name", "Age", "Marks");
    while (fread(&s, sizeof(Student), 1, fp) == 1)
    {
        printf("%-6d %-20s %-6d %-6.2f\n", s.id, s.name, s.age, s.marks);
    }
    printf("----------------\n");

    fclose(fp);
}
void search_student()
{
    int id;
    printf("Enter Student ID to search: ");
    if (scanf("%d", &id) != 1)
    {
        while (getchar() != '\n')
            ;
        printf("Invalid input.\n");
        return;
    }
    while (getchar() != '\n');
    FILE *fp = fopen(DATA_FILE, "rb");
    if (!fp)
    {
        printf("No records found.\n");
        return;
    }
    Student s;
    int found = 0;
    while (fread(&s, sizeof(Student), 1, fp) == 1)
    {
        if (s.id == id)
        {
            printf("\nFound:\nID: %d\nName: %s\nAge: %d\nMarks: %.2f\n", s.id, s.name, s.age, s.marks);
            found = 1;
            break;
        }
    }
    if (!found)
        printf("Student with ID %d not found.\n", id);
    fclose(fp);
}
void update_student()
{
    int id;
    printf("Enter Student ID to update: ");
    if (scanf("%d", &id) != 1)
    {
        while (getchar() != '\n')
            ;
        printf("Invalid input.\n");
        return;
    }
    while (getchar() != '\n')
        ;

    FILE *fp = fopen(DATA_FILE, "rb+");
    if (!fp)
    {
        printf("No records found.\n");
        return;
    }
    Student s;
    int found = 0;
    while (fread(&s, sizeof(Student), 1, fp) == 1)
    {
        if (s.id == id)
        {
            printf("Current Name: %s\nEnter new Name (or press Enter to keep): ", s.name);
            char newname[NAME_LEN];
            read_line(newname, NAME_LEN);
            if (strlen(newname) > 0)
                strncpy(s.name, newname, NAME_LEN);
            printf("Current Age: %d\nEnter new Age (or 0 to keep): ", s.age);
            int newage;
            if (scanf("%d", &newage) != 1)
            {
                while (getchar() != '\n')
                    ;
                newage = 0;
            }
            while (getchar() != '\n')
                ;
            if (newage > 0)
                s.age = newage;

            printf("Current Marks: %.2f\nEnter new Marks (or negative to keep): ", s.marks);
            float newmarks;
            if (scanf("%f", &newmarks) != 1)
            {
                while (getchar() != '\n')
                    ;
                newmarks = -1;
            }
            while (getchar() != '\n')
                ;
            if (newmarks >= 0.0f)
                s.marks = newmarks;

            fseek(fp, -(long)sizeof(Student), SEEK_CUR);
            if (fwrite(&s, sizeof(Student), 1, fp) != 1)
            {
                perror("Update write error");
            }
            else
            {
                printf("Student updated successfully.\n");
            }
            found = 1;
            break;
        }
    }

    if (!found)
        printf("Student with ID %d not found.\n", id);

    fclose(fp);
}
void delete_student()
{
    int id;
    printf("Enter Student ID to delete: ");
    if (scanf("%d", &id) != 1)
    {
        while (getchar() != '\n')
            ;
        printf("Invalid input.\n");
        return;
    }
    while (getchar() != '\n')
        ;

    FILE *fp = fopen(DATA_FILE, "rb");
    if (!fp)
    {
        printf("No records found.\n");
        return;
    }

    FILE *tmp = fopen("temp.dat", "wb");
    if (!tmp)
    {
        perror("Unable to open temporary file");
        fclose(fp);
        return;
    }

    Student s;
    int found = 0;
    while (fread(&s, sizeof(Student), 1, fp) == 1)
    {
        if (s.id == id)
        {
            found = 1;
            continue;
        }
        fwrite(&s, sizeof(Student), 1, tmp);
    }
    fclose(fp);
    fclose(tmp);
    if (found)
    {
        remove(DATA_FILE);
        rename("temp.dat", DATA_FILE);
        printf("Student with ID %d deleted.\n", id);
    }
    else
    {
        remove("temp.dat");
        printf("Student with ID %d not found.\n", id);
    }
}
void menu()
{
    int choice;
    while (1)
    {
        printf("\nStudent Database Menu\n");
        printf("1. Add Student\n");
        printf("2. List Students\n");
        printf("3. Search Student by ID\n");
        printf("4. Update Student by ID\n");
        printf("5. Delete Student by ID\n");
        printf("6. Exit\n");
        printf("Enter choice: ");

        if (scanf("%d", &choice) != 1)
        {
            while (getchar() != '\n')
                ;
            printf("Invalid input.\n");
            continue;
        }
        while (getchar() != '\n')
            ;

        switch (choice)
        {
        case 1:
            add_student();
            break;
        case 2:
            list_students();
            break;
        case 3:
            search_student();
            break;
        case 4:
            update_student();
            break;
        case 5:
            delete_student();
            break;
        case 6:
            printf("Exiting.\n");
            return;
        default:
            printf("Invalid choice. Try again.\n");
        }
    }
}
int main()
{
    menu();
    return 0;
}
