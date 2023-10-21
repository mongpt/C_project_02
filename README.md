# C_project_02

## CSV reader
```
CSV (Comma Separated Values) files are a common way to store tabular data, making them a
valuable format for data interchange between applications. Write a program that reads house
temperature data from a CSV-file and prints a horizontal bar graph of the temperature in the
selected room. The CSV file starts with a header row and then continues with lines of comma
separated data. The header row contains two titles: Room and Temperature. Data lines contain
room name and the temperature separated by commas.
Temperature,Room
22.5,Kitchen
18.7,Living Room
24.2,Bedroom
20.1,Kitchen
12.3,Living Room
23.8,Bedroom
16.9,Kitchen
19.4,Living Room
13.7,Bedroom
21.8,Kitchen
11.5,Living Room
24.9,Bedroom
Program must ask user to select a room and then print the temperatures with one decimal
precision followed with a bar graph using dashes (-). Each dash corresponds to 0.5 centigrade
and temperatures in the range of 0 – 30 are printed as horizontal bars. Temperatures that are
outside or the range are printed as a single X on the left. If the selected room does not exist
then program must print an error message.
For example, (not based on the data above):
Bedroom
20.4 ----------------------------------------
22.5 -------------------------------------------
21.4 ------------------------------------------
21.6 -----------------------------------------
32.3 X
18.2 ------------------------------------
18.8 -------------------------------------
19.5 --------------------------------------
21.3 -----------------------------------------
```
## Code
```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <stdbool.h>

#define ROOM_LEN 20
#define LINE_SIZE 100
#define FILE_NAME "temperatures.csv"

typedef struct node
{
    float temp;
    char room[ROOM_LEN];
} temperature;

temperature *allocateMemory(char *line);
void printBar(float temp);

int main()
{
    FILE *file;
    // Open file
    file = fopen(FILE_NAME, "r");
    if (file == NULL)
    {
        printf("Error opening file %s\n", FILE_NAME);
        return 0;
    }
    // count how many valid data lines in order to declare enough space for weeklySchedule
    int maxLines = 0;
    while (!feof(file))
    {
        char line[LINE_SIZE];
        if (fgets(line, LINE_SIZE, file))
        {
            if (strchr(line, ',') != NULL)
            {
                maxLines++;
            }
        }
    }
    // printf("Total valid data lines: %d\n", maxLines);
    //  define array of pointers to struct
    temperature *allTemps[maxLines];
    // Reset file pointer
    rewind(file);
    // Read file to array
    int totalLines = 0;
    while (!feof(file))
    {
        char line[LINE_SIZE];
        if (fgets(line, LINE_SIZE, file))
        {
            // printf(">>>> Here\n");
            if ((allTemps[totalLines] = allocateMemory(line)) != NULL) // data was OK and a new node has been created
            {
                totalLines++;
            }
        }
    }
    // Close file
    fclose(file);

    // print test data
    for (int i = 0; i < totalLines; i++)
    {
        printf("%s: %.1f\n", allTemps[i]->room, allTemps[i]->temp);
    }
    printf("\n");
    // get a list of rooms
    int countRooms = 0;
    bool dubplicate = false;
    for (int i = 0; i < totalLines - 1; i++)
    {
        for (int j = i + 1; j < totalLines; j++)
        {
            if (strcmp(allTemps[i]->room, allTemps[j]->room) == 0)
            {
                dubplicate = true;
                break;
            }
        }
        if (!dubplicate)
        {
            countRooms++;
        }
        else
        {
            dubplicate = false;
        }
    }
    countRooms++;
    // printf("Unique rooms: %d\n", countRooms);
    //  Create array of unique rooms
    char rooms[countRooms][ROOM_LEN];
    int roomIndex = 0;
    dubplicate = false;
    for (int i = 0; i < totalLines - 1; i++)
    {
        for (int j = i + 1; j < totalLines; j++)
        {
            if (strcmp(allTemps[i]->room, allTemps[j]->room) == 0)
            {
                dubplicate = true;
                break;
            }
        }
        if (!dubplicate)
        {
            strcpy(rooms[roomIndex], allTemps[i]->room);
            roomIndex++;
        }
        else
        {
            dubplicate = false;
        }
    }
    strcpy(rooms[roomIndex], allTemps[totalLines - 1]->room);
    // show list of rooms
    for (int i = 0; i < countRooms; i++)
    {
        printf("%d. %s\n", i + 1, rooms[i]);
    }
    // Ask user for room
    int choice = -1;
    while (choice != 0)
    {
        printf("Select a room or 0 to exit: ");
        while (scanf("%d", &choice) != 1)
        {
            fprintf(stderr, "Invalid input\n");
            printf("Select a room or 0 to exit: ");
            while (getchar() != '\n')
                ;
        }
        if (choice < 0 || choice > countRooms)
        {
            fprintf(stderr, "Room does not exist\n");
        }
        else if (choice == 0)
        {
            break;
        }
        else
        {
            printf("%s\n", rooms[choice - 1]);
            // search and print data
            for (int i = 0; i < totalLines; i++)
            {
                if (strcmp(allTemps[i]->room, rooms[choice - 1]) == 0)
                {
                    printBar(allTemps[i]->temp);
                    printf("\n");
                }
            }
        }
    }
    // free memory
    for (int i = 0; i < totalLines; i++)
    {
        free(allTemps[i]);
    }

    return 0;
}

temperature *allocateMemory(char *line)
{
    float temp;
    char room[ROOM_LEN];
    // Read line and validate input
    if (sscanf(line, "%f,%[^\n]", &temp, room) != 2)
    {
        // printf("Error reading line: %s\n", line);
        return NULL;
    }
    // allocate memory for temperature
    temperature *newNode = malloc(sizeof(temperature));
    if (newNode == NULL)
    {
        printf("Error allocating memory\n");
        free(newNode);
        exit(1);
    }
    // Copy data to struct
    newNode->temp = temp;
    strcpy(newNode->room, room);
    return newNode;
}

void printBar(float temp)
{
    if (temp < 0 || temp > 30)
    {
        printf("%.1f X", temp);
        return;
    }
    int count = temp * 2;
    printf("%.1f ", temp);
    for (int i = 0; i < count; i++)
    {
        printf("-");
    }
}
```
