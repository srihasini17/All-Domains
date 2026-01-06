// crud for any domain

#include <stdio.h>
#include <string.h>
#define MAX_FIELDS 10
#define SIZE_OF_STRING 50
#define FILENAME "records.dat"

int status = 1;
int getFieldCount(void);
int loadFieldsIntoArray(char fields[MAX_FIELDS][SIZE_OF_STRING]);
int splitRecord(char *line, char values[MAX_FIELDS][SIZE_OF_STRING]);
int createRecord(char fields[MAX_FIELDS][SIZE_OF_STRING], int fieldCount);
int showAllRecords(char fields[MAX_FIELDS][SIZE_OF_STRING], int fieldCount);
int updateRecord();
int deleteRecord();
void printMenu();

int main()
{
    int fieldCount = getFieldCount();
    printf("The field count is: %d\n", fieldCount);
    char fields[MAX_FIELDS][SIZE_OF_STRING];
    fieldCount = loadFieldsIntoArray(fields);
    if(fieldCount == 0)
    {
        printf("No fields loaded.");
    }
    else
    {
        printf("\nLoaded files: \n");
        int counter = 0;
        for(counter = 0; counter<fieldCount; counter++)
        {
            printf("%s\n", fields[counter]);
        }   
    }
    int choice;
    while(1) 
    {
        printMenu();
        printf("\nEnter your choice: ");
        scanf("%d", &choice);
        getchar();    
        switch(choice) 
        {
            case 1:
                createRecord(fields, fieldCount);
                break;
            case 2:
                showAllRecords(fields, fieldCount);
                break;
            case 3:
                updateRecord();
                break;
            case 4:
                deleteRecord();
                break;
            case 5:
                printf("Exiting...\n");
                return 0;
            default:
                printf("Invalid choice!\n");
        }
    }
    return 0;

}


int getFieldCount()
{
    FILE *fp;
    fp = fopen("fields.cfg", "r");
    if (fp == NULL)
    {
        perror("File open failed");
        return 0;
    }
    char field[100];
    int fieldCount = 0;

    while(fgets(field, sizeof(field), fp) != NULL)
    {
        fieldCount++;
    }
    fclose(fp);
    return fieldCount;
}

int loadFieldsIntoArray(char fields[MAX_FIELDS][SIZE_OF_STRING])
{
    FILE *fp;
    char line[SIZE_OF_STRING];
    int count = 0;

    fp = fopen("fields.cfg", "r");
    if (fp == NULL)
        return 0;

    while (fgets(line, sizeof(line), fp))
    {
        if (line[0] == '\n')
            continue;

        line[strcspn(line, "\n")] = '\0';
        strcpy(fields[count], line);
        count++;
    }
    fclose(fp);
    return count;
}

int splitRecord(char *line, char values[MAX_FIELDS][SIZE_OF_STRING])
{
    int count = 0;
    char *token = strtok(line, ",");

    while (token != NULL && count < MAX_FIELDS)
    {
        token[strcspn(token, "\n")] = '\0';  // remove newline
        strcpy(values[count], token);
        count++;
        token = strtok(NULL, ",");
    }
    return count;
}


void removeNewLine(char *string)
{
    char *lastCharacter = &string[strlen(string) - 1];
    if (*lastCharacter == '\n')
        *lastCharacter = '\0';
}

void printMenu()
{
    char printMenu[SIZE_OF_STRING];
    FILE *fpMenu;
    int menuCounter = 1;
    fpMenu = fopen("menu.cfg", "r");
    while(fgets(printMenu, SIZE_OF_STRING, fpMenu))
    {
        removeNewLine(printMenu);
        printf("%d:%s\n", menuCounter, printMenu);
        menuCounter++;
    }
    fclose(fpMenu);
}

int createRecord(char fields[MAX_FIELDS][SIZE_OF_STRING], int fieldCount)
{
    int counter;
    FILE *fp = fopen(FILENAME, "a");
    char input[SIZE_OF_STRING];

    for (counter = 0; counter < fieldCount; counter++)
    {
        
        if (strcmp(fields[counter], "status") == 0)
        {
            fprintf(fp, "ACTIVE");
        }
        else  
        {
            printf("Enter %s: ", fields[counter]);
            fgets(input, SIZE_OF_STRING, stdin);

            if (input[0] == '\n') 
            {
                counter--;
                continue;
            }

            input[strcspn(input, "\n")] = '\0';
            fprintf(fp, "%s", input);
        }

        if (counter < fieldCount - 1)
            fprintf(fp, ",");
        else
            fprintf(fp, "\n");
    }

    fclose(fp);
    printf("Record created successfully!\n");
    return 1;
}

int showAllRecords(char fields[MAX_FIELDS][SIZE_OF_STRING], int fieldCount)
{
    FILE *fp = fopen(FILENAME, "r");
    char line[200];
    char values[MAX_FIELDS][SIZE_OF_STRING];
    int i;

    if (!fp)
    {
        printf("File not found\n");
        return 0;
    }

    printf("\n------ ALL RECORDS ------\n\n");

    while (fgets(line, sizeof(line), fp))
    {
        int count = splitRecord(line, values);

        for (i = 0; i < count; i++)
        {
            printf("%s: %s\n", fields[i], values[i]);
        }
        printf("\n");
    }

    fclose(fp);
    return 1;
}



int deleteRecord()
{
    
}

int updateRecord()
{
    FILE *fp = fopen(FILENAME, "r");
    FILE *temp = fopen("temp.dat", "w");

    char fields[MAX_FIELDS][SIZE_OF_STRING];
    char values[MAX_FIELDS][SIZE_OF_STRING];
    char line[200];
    char searchId[SIZE_OF_STRING];
    int fieldCount, found = 0;
    int i, choice;

    if (!fp || !temp)
    {
        printf("File error\n");
        return 0;
    }

    fieldCount = loadFieldsIntoArray(fields);

    printf("Enter Account ID to update: ");
    fgets(searchId, SIZE_OF_STRING, stdin);
    removeNewLine(searchId);

    while (fgets(line, sizeof(line), fp))
    {
        int count = splitRecord(line, values);

        if (strcmp(values[0], searchId) == 0)
        {
            found = 1;

            printf("\nWhich field do you want to update?\n");
            for (i = 1; i < fieldCount; i++)
            {
                printf("%d. %s\n", i, fields[i]);
            }

            printf("Enter field number: ");
            scanf("%d", &choice);
            getchar();

            if (choice > 0 && choice < fieldCount)
            {
                printf("Enter new value for %s: ", fields[choice]);
                fgets(values[choice], SIZE_OF_STRING, stdin);
                removeNewLine(values[choice]);

                printf("Record updated successfully!\n");
            }
            else
            {
                printf("Invalid field choice\n");
            }
        }

        for (i = 0; i < count; i++)
        {
            fprintf(temp, "%s", values[i]);
            if (i < count - 1)
                fprintf(temp, ",");
            else
                fprintf(temp, "\n");
        }
    }

    fclose(fp);
    fclose(temp);

    remove(FILENAME);
    rename("temp.dat", FILENAME);

    if (!found)
        printf("Account ID not found!\n");

    return 1;
}

