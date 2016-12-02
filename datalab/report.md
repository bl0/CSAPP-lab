## Datalab Report
### Bin Liu (2014013466)
```c
/*
 * bitAnd - x&y using only ~ and |
 *   Example: bitAnd(6, 5) = 4
 *   Legal ops: ~ |
 *   Max ops: 8
 *   Rating: 1
 */
int bitAnd(int x, int y) {
  /* De Morgan's laws */
  return ~(~x | ~y);
}
/*
 * getByte - Extract byte n from word x
 *   Bytes numbered from 0 (LSB) to 3 (MSB)
 *   Examples: getByte(0x12345678,1) = 0x56
 *   Legal ops: ! ~ & ^ | + << >>
 *   Max ops: 6
 *   Rating: 2
 */
int getByte(int x, int n) {
  /* exploit ability of shift to compute 8*n and x >> 8n
     exploit ability of & to get the value of last byte of x >> 8n */
  x = x >> (n << 3);
  return x & 0xFF;
}
/*
 * logicalShift - shift x to the right by n, using a logical shift
 *   Can assume that 0 <= n <= 31
 *   Examples: logicalShift(0x87654321,4) = 0x08765432
 *   Legal ops: ~ & ^ | + << >>
 *   Max ops: 20
 *   Rating: 3
 */
int logicalShift(int x, int n) {
  /* user ~0U to get -1 */
  /* use 1 << n - 1 to get 00..0011111(n zeros) */
  /* use (x >> n) & mask to get answer*/
  int m = ~n + 1 + 32;
  int mask = (((m >> 5) ^ 1) << (m & 31)) + (~0);
  x = x >> n;
  return (x & mask);
}
/*
 * bitCount - returns count of number of 1's in word
 *   Examples: bitCount(5) = 2, bitCount(7) = 3
 *   Legal ops: ! ~ & ^ | + << >>
 *   Max ops: 40
 *   Rating: 4
 */
int bitCount(int x) {
  /* Idea:
        Divide and Conquer
     Implementation:
        1. use left shift to get big const masks.
        2. use mask to count the number of 1 in neighbor.
        */
  int c = 0; // counter
  int x5555 = (0x55 << 8) + 0x55;
  int x3333 = (0x33 << 8) + 0x33;
  int x0f0f = (0x0f << 8) + 0x0f;
  int x55555555 = (x5555 << 16) + x5555; // = 0x5555 5555
  int x33333333 = (x3333 << 16) + x3333; // = 0x3333 3333
  int x0f0f0f0f = (x0f0f << 16) + x0f0f; // = 0x0f0f 0f0f
  int x00ff00ff = (0xff << 16) + 0xff;   // = 0x00ff 00ff
  int x0000ffff = (0xff << 8) + 0xff;    // = 0x0000 ffff

  c = (x & x55555555) + ((x >> 1) & x55555555);
  c = (c & x33333333) + ((c >> 2) & x33333333);
  c = (c & x0f0f0f0f) + ((c >> 4) & x0f0f0f0f);
  c = (c & x00ff00ff) + ((c >> 8) & x00ff00ff);
  c = (c & x0000ffff) + ((c >> 16)& x0000ffff);
  return c;
}
/*
 * bang - Compute !x without using !
 *   Examples: bang(3) = 0, bang(0) = 1
 *   Legal ops: ~ & ^ | + << >>
 *   Max ops: 12
 *   Rating: 4
 */
int bang(int x) {
  /* Idea:
        Change all the value except 0 to positive,
        and then subtract 1. Finally the sign bit is the answer
     Implementation:
        use 1 << 31 + ~0u to get 011...11(31 ones)
        user (x >> 1) & 01111 to get logical shift x to the right by 1
        user high31 | (x&1) to make all the value except 0 to positive
     */
  int mask = (1 << 31) + (~0);
  int high31 = (x >> 1) & mask;
  int tmp = (high31 | (x&1)) + ~0;
  return (tmp >> 31) & 1;
}
/*
 * tmin - return minimum two's complement integer
 *   Legal ops: ! ~ & ^ | + << >>
 *   Max ops: 4
 *   Rating: 1
 */
int tmin(void) {
    /* According to the rule of two's complement,
     *  the mininum is 0x8fffffff
     */
  return 1 << 31;
}
/*
 * fitsBits - return 1 if x can be represented as an
 *  n-bit, two's complement integer.
 *   1 <= n <= 32
 *   Examples: fitsBits(5,3) = 0, fitsBits(-4,3) = 1
 *   Legal ops: ! ~ & ^ | + << >>
 *   Max ops: 15
 *   Rating: 2
 */
int fitsBits(int x, int n) {
  int x1 = x ^ (x >> 31);
  return !(x1 >> (n + ~0));
}
/*
 * divpwr2 - Compute x/(2^n), for 0 <= n <= 30
 *  Round toward zero
 *   Examples: divpwr2(15,1) = 7, divpwr2(-33,4) = -2
 *   Legal ops: ! ~ & ^ | + << >>
 *   Max ops: 15
 *   Rating: 2
 */
int divpwr2(int x, int n) {
  // bias = 0..0011..11(n ones) if x > 0 else = 0000
  int mask = (1 << n) ^ ~0;
  int bias = (x >> 31) & mask;
  return (x + bias) >> n;
}
/*
 * negate - return -x
 *   Example: negate(1) = -1.
 *   Legal ops: ! ~ & ^ | + << >>
 *   Max ops: 5
 *   Rating: 2
 */
int negate(int x) {
  /* According to the rule of two's complement
   *   -x = ~x + 1 */
  return ~x + 1;
}
/*
 * isPositive - return 1 if x > 0, return 0 otherwise
 *   Example: isPositive(-1) = 0.
 *   Legal ops: ! ~ & ^ | + << >>
 *   Max ops: 8
 *   Rating: 3
 */
int isPositive(int x) {
  /* use !(x >> 31) to determine if x is not negtive
   * use !x to determine if x is zero
   * x is positive = x is not negtive and x is not zero
   */
  return !(x >> 31) & (!(!x));
}
/*
 * isLessOrEqual - if x <= y  then return 1, else return 0
 *   Example: isLessOrEqual(4,5) = 1.
 *   Legal ops: ! ~ & ^ | + << >>
 *   Max ops: 24
 *   Rating: 3
 */
int isLessOrEqual(int x, int y) {
  /* Idea:
   *    x <= y is equivalent to y - x >= 0 when x and y have same sign.
   *    When x < 0 and y > 0, x < y obviously.
   *    When x > 0 and y < 0, x > y obviously.
   * Implementation:
   *    use (x >> 31) & 1 to get the sign of x. The same with y.
   *    use y + (~x + 1) to get the value of y-x
   *    use following expression to return the right value.
   *      (sub_is_positive | ((!sign_y) & sign_x)) & (sign_x | !sign_y)
   */
  int sign_x = (x >> 31) & 1;
  int sign_y = (y >> 31) & 1;
  int sub = y + (~x + 1);
  int sub_is_positive = !(sub >> 31);
  return (sub_is_positive | ((!sign_y) & sign_x)) & (sign_x | !sign_y);
}
/*
 * ilog2 - return floor(log base 2 of x), where x > 0
 *   Example: ilog2(16) = 4
 *   Legal ops: ! ~ & ^ | + << >>
 *   Max ops: 90
 *   Rating: 4
 */
int ilog2(int x) {
  /*
   * Idea:
   *    Divide and Conquer, Determine if x > 2^16, 2^8, 2^4, 2^2, 2^1 separately.
   * Implementation:
   *    use 1 << n to get 2^n
   *    use x + (~(1 << n) + 1) to get the value of x - 1<<n
   *    use !((x + (~(1 << 16) + 1)) >> 31) to determine if x > 2^n
   *      and save it in gtpn where n = 16, 8, 4, 2, 1]
   *    if x > 2^n, then make x = x - 2^n. add n over result.
   */
  int gtp16 = !((x + (~(1 << 16) + 1)) >> 31);
  int mask1 = gtp16 << 4;
  int x1 = x >> mask1;

  int gtp8 = !((x1 + (~(1 << 8) + 1)) >> 31);
  int mask2 = gtp8 << 3;
  int x2 = x1 >> mask2;

  int gtp4 = !((x2 + (~(1 << 4) + 1)) >> 31);
  int mask3 = gtp4 << 2;
  int x3 = x2 >> mask3;

  int gtp2 = !((x3 + (~(1 << 2) + 1)) >> 31);
  int mask4 = gtp2 << 1;
  int x4 = x3 >> mask4;

  int gtp1 = !((x4 + (~(1 << 1) +1)) >> 31);
  int mask5 = gtp1 << 0;

  return mask1 + mask2 + mask3 + mask4 + mask5;
}
/*
 * float_neg - Return bit-level equivalent of expression -f for
 *   floating point argument f.
 *   Both the argument and result are passed as unsigned int's, but
 *   they are to be interpreted as the bit-level representations of
 *   single-precision floating point values.
 *   When argument is NaN, return argument.
 *   Legal ops: Any integer/unsigned operations incl. ||, &&. also if, while
 *   Max ops: 10
 *   Rating: 2
 */
unsigned float_neg(unsigned uf) {
  /* Use uf & 0x7F800000 == 0x7F800000 to determine whether the exp part is all 1.
   * Use uf & 0x7FFFFF to determine if the frac part is not all 0.
   * So, if the exp part is all 1 and frac part is not all 0, then uf is Nan.
   * In this case, we just return uf itself.
   * Otherwise, return uf ^ (1 << 31) to reverse the highest bit.
   */
  if (((uf & 0x7F800000) == 0x7F800000) && (uf & 0x7FFFFF))
    return uf;
  return uf ^ (1 << 31);
}
/*
 * float_i2f - Return bit-level equivalent of expression (float) x
 *   Result is returned as unsigned int, but
 *   it is to be interpreted as the bit-level representation of a
 *   single-precision floating point values.
 *   Legal ops: Any integer/unsigned operations incl. ||, &&. also if, while
 *   Max ops: 30
 *   Rating: 4
 */
unsigned float_i2f(int x) {
  /* Idea:
   *    Calculat the sign, frac, exp part and put them together.
   *    sign = 0 if x > 0 else 1
   *    exp  = 127 + floor(log2(x)) + (0 or 1), depending on the situation.
   * Implentation:
   *    Refer to commentary in code.
   */

  int neg = 0;  // to store if x < 0
  int exp = 0;  // to store the exp part
  int i = 30;   // loop counter
  int frac = 0; // frac part
  int remain = 0; // part that is rounded.

  // Special case.
  if (x == 0x80000000)
    return 0xcf000000;
  if (x == 0)
    return 0;
  if (x >> 31) {
    neg = 1;
    x = -x;
  } // Now, x is positive

  // Substract i by 1 until x >> i is not zero,
  // so that i is the highest bit of 1 = floor(log2(x)).
  while (!(x >> i)) {
    i--;
  }

  // Get exp, 127 = bias = 2^(8-1) - 1.
  exp = i + 127;

  // Shift x left to remove all the zeros in the front of x.
  x = (x << (31 - i));

  // Get the frac part. 0x7fffff = 1 << 23 - 1.
  frac = 0x7fffff & (x >> 8);

  // Get the remain part which will be rounded.
  remain = x & 0xff;

  /* Determine if frac should be added by 1,
   *    according to the rule of Round-to-even in IEEE. */
  frac += (remain > 0x80) || ((remain == 0x80) && (frac & 1));

  // If frac is too biger for 23 bits, adjust the value and exp
  if(frac >= 0x800000){
    frac &= 0x7fffff;
    exp += 1;
  }

  // Put neg, exp, frac together and return the answer.
  return (neg << 31) | (exp << 23) | (frac);
}
/*
 * float_twice - Return bit-level equivalent of expression 2*f for
 *   floating point argument f.
 *   Both the argument and result are passed as unsigned int's, but
 *   they are to be interpreted as the bit-level representation of
 *   single-precision floating point values.
 *   When argument is NaN, return argument
 *   Legal ops: Any integer/unsigned operations incl. ||, &&. also if, while
 *   Max ops: 30
 *   Rating: 4
 */
unsigned float_twice(unsigned uf) {
  /* Idea:
   *    If uf is Inf or Nan, return uf it self.
   *    If uf is normalized, then we just need to add 1 to exp part of x.
   *    Else if uf is denormalized, E = 1 - bias, M = f
   *      If f is less than 0.5, then if we multiply f by 2,
   *        the new number is also denormalized.
   *      If f is noe less than 0.5, then if we multiply f by 2,
   *        the new number will be normaiized,
   *        and f' will be 2f - 1 and exp' will be 1.
   *        so the new value will be (f'+1) * 2^(exp'-bias) = (2f)*2^(1-bias) = 2*uf
   *      In summary, we just need to shift the frac part left by 1.
   */
  // INF and　Nan
  if ((uf & 0x7F800000) == 0x7F800000) {
    return uf;
  }
  // Denormalized case.
  if ((uf & 0x7F800000) == 0) {
    return (uf & 0x80000000) + ((uf & 0x7FFFFF) << 1);
  }
  // Normalized case.
  return uf + 0x800000;
}
```