```c
#include <stdio.h>

int main() {
	int user[3] = { 0, };
	int ans[3] = { 0, };
	int s = 0;
	int b = 0;
	scanf_s("%d %d %d", &user[0], &user[1], &user[2]);
	scanf_s("%d %d %d", &ans[0], &ans[1], &ans[2]);
	for (int i = 0; i < 3; i++) {
		for (int j = 0; j < 3; j++) {
			if (user[i] == ans[j]) {
				if (i == j) {
					s++;
				}
				else {
					b++;
				}
			}
		}
	}
	if (s == 3) {
		printf("HOMERUN!\n");
	}
	printf("%d ball %d strike", b, s);
}
```
<img width="922" height="182" alt="image" src="https://github.com/user-attachments/assets/e19268eb-1cc4-49f9-987d-d19d19997dae" />
