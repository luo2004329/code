# code#include <stdio.h>
#include <stdlib.h>
#include <string.h>

// 定義學生結構體
typedef struct {
    char ID[11];
    char name[11];
    char gender[3];
    char phone[11];
    float score;
} Student;

// 檢查學號是否已存在
int isIDExist(const char *ID) {
    FILE *file = fopen("student.dat", "rb");
    if (!file) {
        return 0; // 檔案不存在，表示學號不存在
    }

    Student stu;
    while (fread(&stu, sizeof(Student), 1, file)) {
        if (strcmp(stu.ID, ID) == 0) {
            fclose(file);
            return 1; // 學號已存在
        }
    }

    fclose(file);
    return 0; // 學號不存在
}

// 添加學生資料到檔案
void addStudents() {
    FILE *file = fopen("student.dat", "ab");
    if (!file) {
        printf("無法開啟檔案！\n");
        return;
    }

    Student stu;
    char choice;

    do {
        printf("\n");
        printf("╔══════════════════════════╗\n");
        printf("║        新增學生資料      ║\n");
        printf("╚══════════════════════════╝\n");
        printf("學號: ");
        scanf("%10s", stu.ID);

        if (isIDExist(stu.ID)) {
            printf("該學號已存在，不允許新增。\n");
        } else {
            printf("姓名: ");
            scanf("%10s", stu.name);
            printf("性別: ");
            scanf("%2s", stu.gender);
            printf("電話: ");
            scanf("%10s", stu.phone);
            printf("學期成績: ");
            scanf("%f", &stu.score);

            fwrite(&stu, sizeof(Student), 1, file);

            printf("學生資料已新增。\n");
        }

        printf("是否繼續新增學生資料？ (y/n): ");
        getchar(); // 吸收換行符
        choice = getchar();

        // 返回主選單
        printf("返回主選單...\n");

    } while (choice == 'y' || choice == 'Y');

    fclose(file);
}


// 修改學生資料
void modifyStudent() {
    FILE *file = fopen("student.dat", "rb+");
    if (!file) {
        printf("無法開啟檔案！\n");
        return;
    }

    char modifyID[11];
    printf("\n");
    printf("╔══════════════════════════╗\n");
    printf("║        修改學生資料      ║\n");
    printf("╚══════════════════════════╝\n");
    printf("輸入要修改的學號 (輸入0返回主選單): ");
    scanf("%10s", modifyID);
    
    if (strcmp(modifyID, "0") == 0) {
        fclose(file);
        printf("返回主選單...\n");
        return;
    }

    Student stu;
    int found = 0;
    while (fread(&stu, sizeof(Student), 1, file)) {
        if (strcmp(stu.ID, modifyID) == 0) {
            printf("學號: %s\n姓名: %s\n性別: %s\n電話: %s\n學期成績: %.2f\n", 
                   stu.ID, stu.name, stu.gender, stu.phone, stu.score);
            printf("輸入新的姓名: ");
            scanf("%10s", stu.name);
            printf("輸入新的性別: ");
            scanf("%2s", stu.gender);
            printf("輸入新的電話: ");
            scanf("%10s", stu.phone);
            printf("輸入新的學期成績: ");
            scanf("%f", &stu.score);

            fseek(file, -sizeof(Student), SEEK_CUR);
            fwrite(&stu, sizeof(Student), 1, file);
            found = 1;
            break;
        }
    }

    if (!found) {
        printf("找不到該學號的學生資料。\n");
    } else {
        printf("學生資料已修改。\n");
    }
    fclose(file);

    // 返回主選單
    printf("返回主選單...\n");
}

// 刪除學生資料
void deleteStudent() {
    FILE *file = fopen("student.dat", "rb");
    if (!file) {
        printf("無法開啟檔案！\n");
        return;
    }

    char deleteID[11];
    printf("\n");
    printf("╔══════════════════════════╗\n");
    printf("║        刪除學生資料      ║\n");
    printf("╚══════════════════════════╝\n");
    printf("輸入要刪除的學號 (輸入0返回主選單): ");
    scanf("%10s", deleteID);
    
    if (strcmp(deleteID, "0") == 0) {
        fclose(file);
        printf("返回主選單...\n");
        return;
    }

    FILE *tempFile = fopen("temp.dat", "wb");
    if (!tempFile) {
        printf("無法創建臨時檔案！\n");
        fclose(file);
        return;
    }

    Student stu;
    int found = 0;
    while (fread(&stu, sizeof(Student), 1, file)) {
        if (strcmp(stu.ID, deleteID) != 0) {
            fwrite(&stu, sizeof(Student), 1, tempFile);
        } else {
            found = 1;
        }
    }

    fclose(file);
    fclose(tempFile);

    if (found) {
        remove("student.dat");
        rename("temp.dat", "student.dat");
        printf("學生資料已刪除。\n");
    } else {
        remove("temp.dat");
        printf("找不到該學號的學生資料。\n");
    }

    // 返回主選單
    printf("返回主選單...\n");
}

// 查詢學生資料
void searchStudent() {
    FILE *file = fopen("student.dat", "rb");
    if (!file) {
        printf("無法開啟檔案！\n");
        return;
    }

    char searchID[11];
    printf("\n");
    printf("╔══════════════════════════╗\n");
    printf("║        查詢學生資料      ║\n");
    printf("╚══════════════════════════╝\n");
    printf("輸入要查詢的學號 (輸入0返回主選單): ");
    scanf("%10s", searchID);
    
    if (strcmp(searchID, "0") == 0) {
        fclose(file);
        printf("返回主選單...\n");
        return;
    }

    Student stu;
    int found = 0;
    while (fread(&stu, sizeof(Student), 1, file)) {
        if (strcmp(stu.ID, searchID) == 0) {
            printf("學號: %s\n姓名: %s\n性別: %s\n電話: %s\n學期成績: %.2f\n", 
                   stu.ID, stu.name, stu.gender, stu.phone, stu.score);
            found = 1;
            break;
        }
    }

    if (!found) {
        printf("找不到該學號的學生資料。\n");
    }
    fclose(file);

    // 返回主選單
    printf("返回主選單...\n");
}


// 列出所有學生資料
void listStudents() {
    FILE *file = fopen("student.dat", "rb");
    if (!file) {
        printf("無法開啟檔案！\n");
        return;
    }

    Student stu;
    printf("學號\t姓名\t性別\t電話\t學期成績\n");
    printf("------------------------------------------------\n");
    while (fread(&stu, sizeof(Student), 1, file)) {
        printf("%s\t%s\t%s\t%s\t%.2f\n", 
               stu.ID, stu.name, stu.gender, stu.phone, stu.score);
    }
    fclose(file);
}


// 按學號比較兩個學生的函數（由小到大）
int compareByIDAsc(const void *a, const void *b) {
    return strcmp((*(Student *)a).ID, (*(Student *)b).ID);
}

// 按學號比較兩個學生的函數（由大到小）
int compareByIDDesc(const void *a, const void *b) {
    return strcmp((*(Student *)b).ID, (*(Student *)a).ID);
}

// 按學號排序學生的函數
void sortStudentsByID() {
	int i = 0;
    FILE *file = fopen("student.dat", "rb");
    if (!file) {
        printf("無法開啟檔案！\n");
        return;
    }

    Student students[100]; // 假設最多100個學生
    int count = 0;
    while (fread(&students[count], sizeof(Student), 1, file)) {
        count++;
    }

    int sortChoice;
    printf("排序方式\n");
    printf("1. 由小到大\n");
    printf("2. 由大到小\n");
    printf("選擇: ");
    scanf("%d", &sortChoice);

    switch (sortChoice) {
        case 1:
            qsort(students, count, sizeof(Student), compareByIDAsc);
            break;
        case 2:
            qsort(students, count, sizeof(Student), compareByIDDesc);
            break;
        default:
            printf("無效選項，請重新選擇。\n");
            fclose(file);
            return;
    }

    printf("學號\t姓名\t性別\t電話\t學期成績\n");
    printf("------------------------------------------------\n");
    for (i = 0; i < count; i++) {
        printf("%s\t%s\t%s\t%s\t%.2f\n", 
               students[i].ID, students[i].name, students[i].gender, students[i].phone, students[i].score);
    }

    fclose(file);
}

// 按成績比較兩個學生的函數（由小到大）
int compareByScoreAsc(const void *a, const void *b) {
    return (*(Student *)a).score - (*(Student *)b).score;
}

// 按成績比較兩個學生的函數（由大到小）
int compareByScoreDesc(const void *a, const void *b) {
    return (*(Student *)b).score - (*(Student *)a).score;
}

// 按成績排序學生的函數
void sortStudentsByScore() {
    int sortChoice;
    int count = 0;
    int i = 0;
    printf("排序方式\n");
    printf("1. 由小到大\n");
    printf("2. 由大到小\n");
    printf("選擇: ");
    scanf("%d", &sortChoice);

    FILE *file = fopen("student.dat", "rb");
    if (!file) {
        printf("無法開啟檔案！\n");
        return;
    }

    Student students[100]; // 假設最多100個學生
    while (fread(&students[count], sizeof(Student), 1, file)) {
        count++;
    }

    switch (sortChoice) {
        case 1:
            qsort(students, count, sizeof(Student), compareByScoreAsc);
            break;
        case 2:
            qsort(students, count, sizeof(Student), compareByScoreDesc);
            break;
        default:
            printf("無效選項，請重新選擇。\n");
            fclose(file);
            return;
    }

    printf("學號\t姓名\t性別\t電話\t學期成績\n");
    printf("------------------------------------------------\n");
    for (i = 0; i < count; i++) {
        printf("%s\t%s\t%s\t%s\t%.2f\n", 
               students[i].ID, students[i].name, students[i].gender, students[i].phone, students[i].score);
    }

    fclose(file); // 關閉檔案
}


// 按性別比較兩個學生的函數
int compareByGender(const void *a, const void *b) {
    return strcmp((*(Student *)a).gender, (*(Student *)b).gender);
}

// 按性別排序學生的函數
void sortStudentsByGender() {
	int i = 0;
    FILE *file = fopen("student.dat", "rb");
    if (!file) {
        printf("無法開啟檔案！\n");
        return;
    }

    Student students[100]; // 假設最多100個學生
    int count = 0;
    while (fread(&students[count], sizeof(Student), 1, file)) {
        count++;
    }

    qsort(students, count, sizeof(Student), compareByGender);

    printf("學號\t姓名\t性別\t電話\t學期成績\n");
    printf("------------------------------------------------\n");
    for (i = 0; i < count; i++) {
        printf("%s\t%s\t%s\t%s\t%.2f\n", 
               students[i].ID, students[i].name, students[i].gender, students[i].phone, students[i].score);
    }

    fclose(file);
}


void sortMenu() {
    int sortChoice;
    do {
        printf("\n排序選項\n");
        printf("1. 按成績排序\n");
        printf("2. 按學號排序\n");
		printf("3. 按性別排序\n"); 
        printf("0. 返回主菜單\n");
        printf("選擇: ");
        scanf("%d", &sortChoice);

        switch (sortChoice) {
            case 1:
                sortStudentsByScore();
                break;
            case 2:
                sortStudentsByID();
                break;
            case 3:
                sortStudentsByGender(); 
                break;
            case 0:
                printf("返回主菜單。\n");
                break;
            default:
                printf("無效選項，請重新選擇。\n");
        }
    } while (sortChoice != 0);
}



// 主函數
int main() {
    int choice;
    do {
        printf("\n");
        printf("=====================================\n");
        printf("          學生檔案管理系統\n");
        printf("=====================================\n");
        printf("1. 新增學生\n");
        printf("2. 修改學生\n");
        printf("3. 刪除學生\n");
        printf("4. 查詢學生\n");
        printf("5. 列出所有學生\n");
        printf("6. 排序\n"); 
        printf("0. 退出\n");
        printf("-------------------------------------\n");
        printf("請選擇您要執行的操作: ");
        scanf("%d", &choice);

        switch (choice) {
            case 1:
                addStudents();
                break;
            case 2:
                modifyStudent();
                break;
            case 3:
                deleteStudent();
                break;
            case 4:
                searchStudent();
                break;
            case 5:
                listStudents();
                break;
            case 6:
                sortMenu(); // 呼叫排序功能的菜單
                break;
            case 0:
                printf("退出系統。\n");
                break;
            default:
                printf("無效選項，請重新選擇。\n");
        }
    } while (choice != 0);

    return 0;
}
