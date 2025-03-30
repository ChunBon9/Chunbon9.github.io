---
title: CODEGATE2025 Write-up
date: 2025-03-30 19:29:00 +09:00
tags: [ctf, writeup, rev]
---

### C0D3Matr1x

`init_proc`에 특별한 것이 없었기에 바로 `main`으로 넘어갔습니다.  

```C
for ( i = 0; i <= 11; ++i )
  {
    if ( (i & 1) != 0 )
    {
      matrixX[23 - i][i] = 1;
      v3 = 23 - i;
      v4 = 24LL * i;
    }
    else
    {
      matrixX[0][25 * i] = 1;
      v3 = 23 - i;
      v4 = 24 * v3;
    }
    matrixX[0][v3 + v4] = 1;
  }
  __isoc99_scanf("%484[^\n]", input);
  for ( j = 2; j <= 23; ++j )
  {
    for ( k = 2; k <= 23; ++k )
    {
      v5 = cnt++;
      buf[j][k] = input[v5];
    }
  }
  for ( m = 1; m <= 24; ++m )
  {
    for ( n = 1; n <= 24; ++n )
    {
      if ( !buf[m][n] )
        buf[m][n] = aC0d3gat3[(n - 1 + m - 1) % 8];
    }
  }
```

인덱스가 짝수일 때는 우하향 하는 곳에 1을, 반대의 경우 우상향 하는 방향에 1을 채워 넣은 X모양 배열을 만들고 0으로 초기화 되어있는 26X26 배열의 중앙 24X24에 입력을 채워 넣습니다. 그 후 입력의 테두리 부분을 정해진 값으로 패딩합니다. 입력이 없던 칸 역시 패딩합니다.  
```C
  subMatrixSum(buf[0], temp[0]);
  swap(temp[0]);                                
  matrixMuti(temp[0], matrixX[0], v21[0]);     
  matrixMuti(matrixX[0], v21[0], temp[0]);    
  swapReverse(temp[0]);                       
  matrixAdd(temp[0], dword_3220[0], v22[0]);
  matrixMuti(v22[0], dword_4D20[0], v23[0]);    
  matrixAdd2(v23[0], dword_3B20[0], v24[0]);
  matrixAdd2(v23[0], dword_4420[0], v25);
```

strip되어 있긴 했지만 내부를 보면 그냥 행렬 연산이었습니다. 그러니 이름을 저렇게 붙여줬고 따로 설명은 안 하겠습니다. 특이해보이는 `subMatrixSum`, `swap`만 설명하겠습니다. `swapReverse`는 이름을 지은 그대로 `swap`의 반대 과정입니다.  
 
#### subMatrix

```C
__int64 __fastcall subMatrixSum(int *buf, int *v20)
{
  __int64 result; // rax
  int i; // [rsp+14h] [rbp-Ch]
  int j; // [rsp+18h] [rbp-8h]

  for ( i = 0; i <= 23; ++i )
  {
    for ( j = 0; j <= 23; ++j )
    {
      result = j;
      v20[24 * i + j] = buf[26 * i + 53 + j]    // [i+2][j+1]
                      + buf[26 * i + 52 + j]    // [i+2][j]
                      + buf[26 * i + 28 + j]    // [i+1][j+2]
                      + buf[26 * i + 27 + j]    // [i+1][j+1]
                      + buf[26 * i + 26 + j]    // [i+1][j]
                      + buf[26 * i + 2 + j]     // [i][j+2]
                      + buf[26 * i + 1 + j]     // [i][j+1]
                      + buf[26 * i + j]         // [i][j]
                      + buf[26 * i + 54 + j];   // [i+2][j+2]
    }
  }
  return result;
}
```

여기선 26X26 배열을 입력으로 받아서 3X3씩 원소들을 더해서 1칸으로 압축해 24X24 배열로 만들고 있습니다.  

#### swap

```C
__int64 __fastcall sub_1769(_DWORD *a1)
{
  __int64 result; // rax
  int i; // [rsp+Ch] [rbp-Ch]
  int j; // [rsp+10h] [rbp-8h]
  int v4; // [rsp+14h] [rbp-4h]

  for ( i = 0; i <= 11; ++i )
  {
    for ( j = i; ; ++j )
    {
      result = (unsigned int)(23 - i);
      if ( j >= (int)result )
        break;
      v4 = a1[24 * j + i];                      // [j][i]
      a1[24 * j + i] = a1[24 * i + 23 - j];     // [j][i] <-- [i][23-j]
      a1[24 * i + 23 - j] = a1[24 * (23 - j) + 23 - i];// [i][23-j] <-- [23-j][23-i]
      a1[24 * (23 - j) + 23 - i] = a1[24 * (23 - i) + j];// [23-j][23-i] <-- [23-i][j]
      a1[24 * (23 - i) + j] = v4;               // [23-i][j] <-- [j][i]
    }
  }
  return result;
}
```
주석을 통해 볼 수 있는 배열의 모서리 부분의 값들을 반시계 방향으로 서로 바꾸고 있습니다. 다음으로는 반시계 방향의 값들을 바꾸고 모두 바꿨다면 안으로 한 줄씩 좁혀가 마저 바꾸고 있습니다. 그럼 `swapReverse`는 시계 방향이겠죠?

<br>

```C
for ( ii = 0; ii <= 23; ++ii )
    {
    for ( jj = 0; jj <= 23; ++jj )
    {
        if ( v24[ii][jj] != dword_5620[ii][jj] )
        {
        puts("Wrong");
        return 0LL;
        }
    }
}
```

각종 연산 이후엔 값을 비교합니다. 이후에도 코드가 더 있긴한데 검증이후는 알아서 되겠거니 하여 넘어갔습니다.  

이제 입력값을 찾아야하는데 역산의 경우 `subMatrixSum`과 행렬곱이 문제입니다. 행렬곱의 경우 다른 한 쪽이 역행렬이 있어야 가능합니다. 이를 SageMath를 통해 확인해봤는데 있었습니다. 다음으로 `subMatrixSum`인데 1개를 통해 9개를 찾아낼 수는 없습니다. 하지만 temp[0][0]값을 만들 때 쓰이는 값은 000, 0PP, 0PI 입니다. (P: 패딩값, I: 입력값) 그러니 역산이 가능하고 이후는 연쇄적으로 한 개씩 복구가 가능합니다. 이를 수행하도록 하겠습니다.

<br>

```python
#ex.py

import numpy as np
buf = [[0 for i in range(26)] for j in range(26)]
result = [...]
matrixX = [...]
dword_3B20 = [...]
dword_4D20_inv = [...]

def swapInv(M):
    for i in range(12):
        for j in range(i,25):
            if j >= (23 - i): break
            temp = M[j][i]
            M[j][i] = M[i][23-j]
            M[i][23-j] = M[23-j][23-i]
            M[23-j][23-i] = M[23-i][j]
            M[23-i][j] = temp
    return M

def swap(M):
    for i in range(12):
        for j in range(i,25):
            if j >= (23 - i): break
            temp = M[j][i]
            M[j][i] = M[23-i][j]
            M[23-i][j] = M[23-j][23-i]
            M[23-j][23-i] = M[i][23-j]
            M[i][23-j] = temp
    return M

r = np.array(result)
t = np.array(dword_3B20)
r = ((r - t))

t = np.array(dword_4D20_inv)
r = np.dot(r, t)

t = np.array(dword_3220)
r = ((r - t))

r = r.tolist()
r = swap(r)

r = np.array(r)
x = np.array(matrixX)
r = np.dot(x, r)
r = np.dot(r, x)

r = r.tolist()
r = swapInv(r)

code = "C0D3GAT3"
for m in range(1, 25):
    for n in range(1, 25):
        if m == 1 or n == 1 or m == 24 or n == 24:
            buf[m][n] = ord(code[(n-1 + m-1) % 8])

flag = [[0 for i in range(22)] for j in range(22)]

for i in range(22):
    for j in range(22):
        buf[i+2][j+2] = (r[i][j] - (buf[i+2][j+1] + buf[i+2][j] + buf[i+1][j+2] + buf[i+1][j+1] + buf[i+1][j] + buf[i][j+2] + buf[i][j+1] + buf[i][j])) % 0xff
        flag[i][j] = buf[i+2][j+2]

s = b""
for i in range(22):
    for j in range(22):
        s += (flag[i][j]%0xff).to_bytes(1, byteorder="little")
        print(f"{flag[i][j]}", end=" ")
    print()

print(len(s))
f = open("temp.txt", "wb")
f.write(s)
f.close()
```

```
C0DEGATE 1s a gl0ba1 internationa1 hacking d3f3ns3 competition and 5ecurity conference. Held annually since 2008, C0D3GAT3 is known as the Olympics for hackers, wh3re hack3rs and security 3xperts from around the world gath3r t0 c0mpet3 for the title of the w0rld's best hack3r. In addition to fierce competition among tru3 white-hat hackers, a juni0r division is also he1d, s3rv1ng as a p1atform f0r discover1ng talented 1ndividuals 1n th3 fi3ld of cyb3rsecurity. You are good hacker.
```

<figure>
<img src="/codegate-2025-writeup/1.png">
<figcaption>YEAH</figcaption>
</figure>

성공했습니다. 문제가 분석도 역산도 꽤나 간단했는데 행렬곱을 `np.dot(A,B)`로 해야하는데 `(A*B)`로 진행, 패딩하는 문자인 `C0DEMATR1X`에서  `0`을 `O`로 보고 몇 시간 삽질 했습니다. 눈 좀 뜨고 살아야겠습니다.

<br>

### cha's ELF

이건 풀지도 분석도 제대로 못 했지만 알아낸 정보만이라도 적어보겠습니다. 

<details>
<summary>cha's ELF pseudo code</summary>
<div markdown="1">

```C
__int64 __fastcall main(int a1, char **a2, char **a3)
{
  memset(src, 0, sizeof(src));
  __isoc99_scanf(&unk_408398, src);
  v31 = strlen(src);
  v42 = v31 & 0x3F;
  v22 = 0x6E90A5E;
  while ( 1 )
  {
    while ( 1 )
    {
      while ( 1 )
      {
        while ( 1 )
        {
          while ( 1 )
          {
            while ( 1 )
            {
              while ( 1 )
              {
                while ( 1 )
                {
                  while ( 1 )
                  {
                    while ( 1 )
                    {
                      while ( 1 )
                      {
                        while ( 1 )
                        {
                          while ( 1 )
                          {
                            while ( 1 )
                            {
                              while ( 1 )
                              {
                                while ( 1 )
                                {
                                  while ( v22 == 0x91E60A56 )
                                  {             // 2
                                    v4 = 0x34F6B4DA;
                                    if ( (dword_40A174 < 10
                                       && ((((_BYTE)dword_40A1BC - 1) * (_BYTE)dword_40A1BC) & 1) == 0) | ((unsigned __int8)~(dword_40A174 < 10) ^ (unsigned __int8)~(((((_BYTE)dword_40A1BC - 1) * (_BYTE)dword_40A1BC) & 1) == 0)) & 1 )
                                      v4 = 0x7D818A59;
                                    v22 = v4;
                                  }
                                  if ( v22 != 0xA0E98A12 )
                                    break;
                                  printf("\n");
                                  free(ptr);
                                  free(v34);
                                  free(v36);
                                  free(v37);
                                  free(v38);
                                  free(v39);
                                  free(inputBuf);
                                  v20 = 0x76382DB9;
                                  if ( (dword_40A174 < 10 && ((((_BYTE)dword_40A1BC - 1) * (_BYTE)dword_40A1BC) & 1) == 0) | (dword_40A174 < 10) ^ (((((_BYTE)dword_40A1BC - 1) * (_BYTE)dword_40A1BC) & 1) == 0) )
                                    v20 = 0xED640DF4;
                                  v22 = v20;
                                }
                                if ( v22 != 0xA9BE128D )
                                  break;
                                v22 = 0xB991F274;// 8
                              }
                              if ( v22 != 0xB823341F )
                                break;
                              v8 = 0x24A83145;
                              if ( (dword_40A174 < 10 && ((((_BYTE)dword_40A1BC - 1) * (_BYTE)dword_40A1BC) & 1) == 0) | (dword_40A174 < 10) ^ (((((_BYTE)dword_40A1BC - 1) * (_BYTE)dword_40A1BC) & 1) == 0) )
                                v8 = 0x637E75A8;
                              v22 = v8;
                            }
                            if ( v22 != 0xB991F274 )
                              break;
                            v15 = 0xDC1083A7;   // 9
                            if ( qword_40A1E0[2 * v24] )
                              v15 = 0x33B5AFA6;
                            v22 = v15;
                          }
                          if ( v22 != 0xBD2B7105 )
                            break;
                          v22 = 0xD225699D;
                        }
                        if ( v22 != 0xBEBF49E2 )
                          break;
                        v22 = 0xB991F274;
                      }
                      if ( v22 != 0xCB60ACD0 )
                        break;
                      v30 = (void (*)(void))mmap(
                                              0LL,
                                              0x292834D0EE111F24LL
                                            - ((v27 + v26) & ((v27 + v26) ^ 0xFFFFFFFFFFFFF000LL))
                                            - 0x292834D0EE110F24LL
                                            + v26
                                            + v27,
                                              7,
                                              34,
                                              -1,
                                              0LL);
                      v29 = v30;
                      v28 = (char *)v30 + v27;
                      v24 = 0;
                      v22 = 0xCE1ECD98;
                    }
                    if ( v22 != 0xCE1ECD98 )
                      break;
                    v13 = (void (*)(void))mmap( // 7
                                            0LL,
                                            v26 + v27 + (~(v27 + v26) | 0xFFFFFFFFFFFFF000LL) + 4097,
                                            7,
                                            34,
                                            -1,
                                            0LL);
                    v14 = 0xCB60ACD0;
                    v30 = v13;
                    v29 = v13;
                    v28 = (char *)v13 + v27;
                    v24 = 0;
                    if ( (dword_40A174 < 10 && ((((_BYTE)dword_40A1BC - 1) * (_BYTE)dword_40A1BC) & 1) == 0) | ((unsigned __int8)~(dword_40A174 < 10) ^ (unsigned __int8)~(((((_BYTE)dword_40A1BC - 1) * (_BYTE)dword_40A1BC) & 1) == 0)) & 1 )
                      v14 = 0xA9BE128D;
                    v22 = v14;
                  }
                  if ( v22 != 0xCE948240 )
                    break;
                  ++v25;
                  v22 = 0xE83C4F8C;
                }
                if ( v22 != 0xD225699D )
                  break;
                v7 = 0xE75F2CA;                 // 5
                if ( qword_40A1E0[2 * v25] )
                  v7 = 0xB823341F;
                v22 = v7;
              }
              if ( v22 != 0xDC1083A7 )
                break;
              v23 = 0;
              v22 = 0x6546E9C1;                 // 10
              v30();
            }
            if ( v22 != 0xE83C4F8C )
              break;
            v11 = 0xCE948240;
            ++v25;
            if ( (dword_40A174 < 10) ^ (((((_BYTE)dword_40A1BC - 1) * (_BYTE)dword_40A1BC) & 1) == 0) | (dword_40A174 < 10 && ((((_BYTE)dword_40A1BC - 1) * (_BYTE)dword_40A1BC) & 1) == 0) )
              v11 = 0xBD2B7105;
            v22 = v11;
          }
          if ( v22 != 0xE8E969DE )
            break;
          v16 = 0xFCBE6C80;
          if ( (dword_40A174 < 10 && ((((_BYTE)dword_40A1BC - 1) * (_BYTE)dword_40A1BC) & 1) == 0) | ((unsigned __int8)~(dword_40A174 < 10) ^ (unsigned __int8)~(((((_BYTE)dword_40A1BC - 1) * (_BYTE)dword_40A1BC) & 1) == 0)) & 1 )
            v16 = 0x2233CFD2;
          v22 = v16;
        }
        if ( v22 != 0xED640DF4 )
          break;
        v22 = 0xF6B7B262;
      }
      if ( v22 != 0xF088D094 )
        break;
      v19 = 0x76382DB9;                         // 14
      if ( (dword_40A174 < 10 && ((((_BYTE)dword_40A1BC - 1) * (_BYTE)dword_40A1BC) & 1) == 0) | (dword_40A174 < 10) ^ (((((_BYTE)dword_40A1BC - 1) * (_BYTE)dword_40A1BC) & 1) == 0) )
        v19 = 0xA0E98A12;
      v22 = v19;
    }
    if ( v22 == 0xF6B7B262 )
      break;
    switch ( v22 )
    {
      case 0xFCBE6C80:
        ++v24;
        v22 = 0x2233CFD2;
        break;
      case 0x6E90A5E:                           // 1
        v3 = 0xF6B7B262;
        if ( v42 == 1 )                         // 길이가 65면 통과
          v3 = 0x91E60A56;
        v22 = v3;
        break;
      case 0xE75F2CA:                           // 6
        v12 = 0xCB60ACD0;
        if ( (dword_40A174 < 10 && ((((_BYTE)dword_40A1BC - 1) * (_BYTE)dword_40A1BC) & 1) == 0) | (dword_40A174 < 10) ^ (((((_BYTE)dword_40A1BC - 1) * (_BYTE)dword_40A1BC) & 1) == 0) )
          v12 = 0xCE1ECD98;
        v22 = v12;
        break;
      case 0x2233CFD2:
        v17 = 0xFCBE6C80;
        ++v24;
        if ( (dword_40A174 < 10) ^ (((((_BYTE)dword_40A1BC - 1) * (_BYTE)dword_40A1BC) & 1) == 0) | (dword_40A174 < 10 && ((((_BYTE)dword_40A1BC - 1) * (_BYTE)dword_40A1BC) & 1) == 0) )
          v17 = 0xBEBF49E2;
        v22 = v17;
        break;
      case 0x24A83145:
        v27 += dest[3 * qword_40A1E0[2 * v25] + 1];
        v26 += dest[3 * qword_40A1E0[2 * v25] + 2];
        v22 = 0x637E75A8;
        break;
      case 0x2D2F62FD:                          // 4
        v22 = 0xD225699D;
        break;
      case 0x33B5AFA6:
        ((void (__fastcall *)(void (*)(void), char *, _QWORD))dest[3 * qword_40A1E0[2 * v24]])(
          v29,
          v28,
          qword_40A1E0[2 * v24 + 1]);
        v29 = (void (*)(void))((char *)v29 + dest[3 * qword_40A1E0[2 * v24] + 1]);
        v28 += dest[3 * qword_40A1E0[2 * v24] + 2];
        v22 = 0xE8E969DE;
        break;
      case 0x34F6B4DA:
        inputBuf = (char *)malloc(v31 + 1);
        strncpy(inputBuf, src, v31);
        inputBuf[v31] = 0;
        v39 = (void *)sub_403BC0();
        v38 = (void *)sub_403DE0();
        v37 = (void *)sub_403E20(v39);
        v36 = (void *)sub_404210(v39, inputBuf, v31);
        v34 = sub_4055E0((__int64)v39);
        ptr = (void *)sub_404C10((__int64)v39, (__int64)inputBuf, v31);
        sub_405670(v39, v38, inputBuf, v31 - 1);
        sub_406AA0();
        v25 = 0;
        v22 = 0x7D818A59;
        break;
      case 0x462667E3:
        v10 = 0xCE948240;
        if ( (dword_40A174 < 10 && ((((_BYTE)dword_40A1BC - 1) * (_BYTE)dword_40A1BC) & 1) == 0) | (dword_40A174 < 10) ^ (((((_BYTE)dword_40A1BC - 1) * (_BYTE)dword_40A1BC) & 1) == 0) )
          v10 = 0xE83C4F8C;
        v22 = v10;
        break;
      case 0x46D4C8FD:
        v22 = 0x462667E3;
        break;
      case 0x637E75A8:
        v9 = 0x24A83145;
        v27 += dest[3 * qword_40A1E0[2 * v25] + 1];
        v26 += dest[3 * qword_40A1E0[2 * v25] + 2];
        if ( (dword_40A174 < 10) ^ (((((_BYTE)dword_40A1BC - 1) * (_BYTE)dword_40A1BC) & 1) == 0) | (dword_40A174 < 10 && ((((_BYTE)dword_40A1BC - 1) * (_BYTE)dword_40A1BC) & 1) == 0) )
          v9 = 0x46D4C8FD;
        v22 = v9;
        break;
      case 0x6546E9C1:
        v18 = 0xF088D094;                       // 11
        if ( v23 < v31 - 1 )
          v18 = 0x76BDBF78;
        v22 = v18;
        break;
      case 0x6F6C087E:
        ++v23;
        v22 = 0x6546E9C1;
        break;
      case 0x76382DB9:
        printf("\n");                           // 15
        free(ptr);
        free(v34);
        free(v36);
        free(v37);
        free(v38);
        free(v39);
        free(inputBuf);
        v22 = 0xA0E98A12;
        break;
      case 0x76BDBF78:
        v22 = 0x6F6C087E;                       // 12
        printf("%02x", ~(~inputBuf[v23] | 0xFFFFFF00));
        break;
      default:                                  // 3
        inputBuf = (char *)malloc(v31 + 1);
        strncpy(inputBuf, src, v31);            // 입력 복사
        inputBuf[v31] = 0;
        v39 = (void *)sub_403BC0();
        v38 = (void *)sub_403DE0();
        v37 = (void *)sub_403E20(v39);
        v36 = (void *)sub_404210(v39, inputBuf, v31);
        v34 = sub_4055E0((__int64)v39);
        ptr = (void *)sub_404C10((__int64)v39, (__int64)inputBuf, v31);
        sub_405670(v39, v38, inputBuf, v31 - 0x457B146F962C8A30LL + 0x457B146F962C8A2FLL);
        sub_406AA0();
        v5 = 0x34F6B4DA;
        v25 = 0;
        v6 = (((_BYTE)dword_40A1BC - 18 + 17) * (_BYTE)dword_40A1BC) & 1;
        if ( (dword_40A174 < 10 && v6 == 0) | (dword_40A174 < 10) ^ (v6 == 0) )
          v5 = 0x2D2F62FD;
        v22 = v5;
        break;
    }
  }
  return 0LL;
}
```
          
</div>
</details>

이걸 보고 한 것은 양수, 음수로 되어 있는 것들을 16진수로 바꿔 어디서 어디로 가는 지 실행흐름을 대강 봤고, 입력이 "111"일 때 어디 어디를 거치는지 주석으로 숫자를 매겨봤습니다.  

더 진행해야할 것으론 `main`이외의 함수들 내부에서도 어떻게 분기 되는가, hooking을 통해 실행시점에서 결정되는 메모리의 값 알아내서 정확한 분기 알아내기, 이를 바탕으로 input값이 어떻게 계산되는지 알아내고 역산하기를 해야할 것 같습니다.
