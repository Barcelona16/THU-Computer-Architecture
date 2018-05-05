1

时空图:

000101010120020002
000000000200200020
000000002002000200
011111111000000000
101010120020002000

吞吐量: 7/18 = 0.39/dt

加速比: (4 * 7) / 18 = 1.56

效率: 28 / (5 * 18) = 0.31

2

禁止表(1, 3, 4, 8)

c0 = 10001101
c1 = 10101111
c2 = 10001111

c0->c0 (5, 7)
c0->c1 (2)
c0->c2 (6)
c1->c0 (5, 7)
c2->c0 (5, 7)

最优调度策略是c0->c1->c0

吞吐量: 2 / (2 + 5) = 0.286dt

6个任务时

吞吐量: 6 / ((2 + 5) * 3 + 9 - 5) = 0.24dt

3

禁止表(1, 3, 6)

c0 = 0100101
c1 = 0101101
c2 = 0100111
c3 = 0101111

c0->c1 (2)
c0->c2 (4)
c0->c0 (5, 7)
c1->c3 (2)
c1->c0 (5, 7)
c2->c2 (4)
c2->c0 (5, 7)
c3->c0 (5, 7)

调度策略:

不等时间间隔: c0->c1->c3->c0 (2, 2, 5)
吞吐量: 3 / 9 = 0.33dt

等时间间隔: c0->c2->c2 (4)
吞吐量 1 / 4 = 0.25dt

10个任务时：
不等间隔:
吞吐量: 10 / (9 * 3 + 7) = 0.294dt
加速比: 10 * 7 / (9 * 3 + 7) = 2.06
等间隔:
吞吐量: 10 / (4 * 9 + 7) = 0.233dt
加速比: 10 * 7 / (4 * 9 + 7) = 1.63

4

LW R1 0(R2)
DADDIU R1, R1, 1
SW R1, 0(R2)
DADDIU R2, R2, 4
DSUB R4, R3, R2
BNEZ R4, LOOP

循环次数 396/4 = 99

(1)

时空图:
       1  2  3  4  5  6  7  8  9  10 11 12 13 14 15 16 17 18
LW     IF ID EX ME WB
DADDIU 0  IF ID S  S  EX ME WB
SW     0  0  IF S  S  ID S  S  EX ME WB
DADDIU 0  0  0  S  S  IF S  S  ID EX ME WB
DSUB   0  0  0  S  S  0  S  S  IF ID S  S  EX ME WB
BNEZ   0  0  0  S  S  0  S  S  0  IF S  S  ID S  S  EX ME WB
       0  0  0  0  0  0  0  0  0  0  0  0  0  0  0  0  IF

每个循环16周期
共需要98 * 16 + 18 = 1586

(2)

时空图:
       1  2  3  4  5  6  7  8  9  10 11 12 13 14 15 16 17 18
LW     IF ID EX ME WB
DADDIU 0  IF ID S  EX ME WB
SW     0  0  IF S  ID EX ME WB
DADDIU 0  0  0  S  IF ID EX ME WB
DSUB   0  0  0  S  0  IF ID EX ME WB
BNEZ   0  0  0  S  0  0  IF ID EX ME WB
       0  0  0  0  0  0  0  0  0  IF

每个循环9周期
共需要98 * 9 + 11 = 893

(3)

LW R1 0(R2)
DADDIU R1, R1, 1
SW R1, 0(R2)
DADDIU R2, R2, 4
DSUB R4, R3, R2
BNEZ R4, LOOP

LW R1 0(R2)
DADDIU R2, R2, 4
DADDIU R1, R1, 1
DSUB R4, R3, R2
BNZ R4, LOOP
SW R1, -4(R2)

时空图:
       1  2  3  4  5  6  7  8  9  10 11 12 13 14 15 16 17 18
LW     IF ID EX ME WB
DADDIU 0  IF ID EX ME WB
SW     0  0  IF ID EX ME WB
DADDIU 0  0  0  IF ID EX ME WB
DSUB   0  0  0  0  IF ID EX ME WB
BNEZ   0  0  0  0  0  IF ID EX ME WB
       0  0  0  0  0  0  IF

每个循环6个周期
共需要6 * 98 + 10 = 598

5

(1) 需要从Mem进行forward的情况只有LW地址,之后使用其中的内容
所以上一个周期，写入的寄存器，本周期使用相同寄存器，这种情况要走这条旁路。
或上两个周期，写入寄存器，本周起使用相同寄存器，也需要旁路。

(2) stall的情况是上一个周期写入寄存器，本周期使用相同寄存器的情况。
完全旁路的MIPS处理器仅这种情况需要stall

(3)

LW R1 0(R2)
ADDIU R1, R1, 1

