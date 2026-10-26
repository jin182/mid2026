```c
#include <stdio.h>

int main() {
	int n = 0;
	scanf_s("%d", &n);
	char cnt = ' ';
	for (int i = 1; i <= n; i++) {
		for (int j = 0; j < n - i; j++) {
			printf(" ");
		}
		for (int j = 0; j < i * 2 - 1; j++) {
			if (i % 2 == 0) {
				cnt = '#';
			}
			else {
				cnt = '*';
			}
			printf("%c", cnt);
		}
		printf("\n");
	}
}
```
<img width="866" height="222" alt="image" src="https://github.com/user-attachments/assets/90d2e306-3231-4841-9d80-6d7107704394" />
