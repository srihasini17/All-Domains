int load_fields_into_array(char fields[MAX_FIELDS][SIZE_OF_STRING])
{
	FILE *fp;
    char line[SIZE_OF_STRING];
    int count = 0;

    fp = fopen("fields.cfg", "r");
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

void removeNewline(char *string)
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
		removeNewline(printMenu);
		printf("%d:%s\n",menuCounter, printMenu);
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
            printf("Enter %s: ", fields[counter]);
            fgets(input, SIZE_OF_STRING, stdin);
            removeNewline(input);
            status = 1;
            fprintf(fp, "%s", input);
            fprintf(fp, "\n");
    }
    fclose(fp);
    printf("Record created successfully!\n");
    return 1;
}


int showAllRecords()
{
	FILE *fp;
	char line[SIZE];
	fp = fopen(FILENAME, "r");
	printf("\n\nAll Records\n\n");
	while(fgets(line, sizeof(line), fp))
	{
		printf("%s", line);
	}
	fclose(fp);
	return 1;
}

int deleteRecord()
{
    FILE *fp = fopen(FILENAME, "r+");
    char line[SIZE];
    char deleteValue[SIZE_OF_STRING];
    //long pos;

    printf("Enter value to delete: ");
    fgets(deleteValue, SIZE_OF_STRING, stdin);

    while (fgets(line, sizeof(line), fp))
    {
        if (strcmp(line, deleteValue) == 0)
        {

            //pos = ftell(fp) - strlen(line) - 1;
            fseek(fp,-(long)sizeof(line), SEEK_CUR);
            status = 0;
            fprintf(fp, "%s\n", (int)strlen(line),status);
            printf("Record deleted successfully!\n");
            fclose(fp);
            return 1;
        }
    }

    printf("Record not found!\n");
    fclose(fp);
    return 0;
}


int updateRecord()
{
    char field[SIZE];
    FILE *fp;
    fp = fopen(FILENAME, "r+");
    char oldValue[SIZE_OF_STRING];
    char newValue[SIZE_OF_STRING];
    //long pos;

    printf("Enter value to update: ");
    fgets(oldValue, SIZE_OF_STRING, stdin);
    removeNewline(oldValue);

    printf("Enter new value: ");
    fgets(newValue, SIZE_OF_STRING, stdin);
    removeNewline(newValue);

    while (fgets(field, sizeof(field), fp))
    {

        if (strcmp(field, oldValue) == 0)
        {
            //pos = ftell(fp) - strlen(line) - 1;
            fseek(fp, -(long)sizeof(field), SEEK_CUR);
            fprintf(fp, "%s\n", newValue);
            printf("Record updated successfully!\n");
            fclose(fp);
            return 1;
        }
    }

    //printf("Record not found!\n");
    fclose(fp);
    return 0;
}
