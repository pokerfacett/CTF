跟前面那道一样，不过是GF(2**128)上，也涉及更高次的幂运算。

查看nextrand首先发现tmp1后两位必然为`0,1`，前两位找规律后数学归纳可得`n`次结果为`A^n,B*(A^(n-1)+...+1)`，后者可以重写成`B*(A^n-1)*(A-1)^-1`，所以事实上只有`A^n`一个未知元，根据两次产生的key1,key2可以解出tmp1。

开始没想明白域的大小，然后被`swordfeng`提醒`2^128-1`这个数smooth，于是离散对数可解，根据tmp1可以还原`N`。

GF(2**128)上的运算很不熟练，幸好sage支持得好，省去不少工作。

由于sage上直接乘法有点问题，所以转成整数进行，该部分的python代码如下：
```
P = 0x100000000000000000000000000000087
A = 0xc6a5777f4dc639d7d1a50d6521e79bfd
B = 0x2e18716441db24baf79ff92393735345

def mul(m, k):
    res = 0
    for i in bin(k)[2:]:
        res = res << 1;
        if (int(i)):
            res = res ^ m
        if (res >> 128):
            res = res ^ P
    return res

def dopow(a,n):
    tmp2 = a
    tmp1 = 1
    s = n
    while s:
        if s % 2:
            tmp1 = mul(tmp2, tmp1)
        tmp2 = mul(tmp2, tmp2)
        s = s / 2
    return tmp1

def str2num(s):
    return int(s.encode('hex'), 16)

def num2str(n, block=16):
    s = hex(n)[2:].strip('L')
    s = '0' * ((32-len(s)) % 32) + s
    return s.decode('hex')

def parse(x):
    res=''
    cnt=0
    while x>0:
        if x%2 == 1:
            if cnt==0:
                t='1'
            elif cnt==1:
                t='x'
            else:
                t='x^%d'%cnt
            res = t+'+'+res
        x /= 2
        cnt+=1
    return res[:-1]

def unparse(s):
    res=0
    t=s.split('+')
    for tt in t:
        tt=str.strip(tt)
        if tt=='1':
            res+=1
        elif tt=='x':
            res+=2
        else:
            a=int(tt[2:])
            res+=(2**a)
    return res

print parse(A)
print parse(B)
print parse(P)
print '---'
tmp=parse(A)
assert unparse(tmp)==A
inv1='x^127 + x^126 + x^125 + x^124 + x^122 + x^119 + x^113 + x^112 + x^109 + x^107 + x^103 + x^102 + x^101 + x^100 + x^99 + x^98 + x^96 + x^92 + x^91 + x^90 + x^87 + x^85 + x^84 + x^78 + x^75 + x^72 + x^59 + x^58 + x^56 + x^54 + x^53 + x^50 + x^46 + x^45 + x^44 + x^42 + x^40 + x^38 + x^37 + x^36 + x^33 + x^31 + x^30 + x^29 + x^28 + x^27 + x^25 + x^23 + x^22 + x^14 + x^13 + x^11 + x^9 + x^8 + x^4 + x^3'
print parse(mul(unparse(inv1),B))
print '---'
plain="One-Time Pad is used here. You won't know that the flag is flag{%s}." % ('a'*16)
cipher='0da8e9e84a99d24d0f788c716ef9e99cc447c3cf12c716206dee92b9ce591dc0722d42462918621120ece68ac64e493a41ea3a70dd7fe2b1d116ac48f08dbf2b26bd63834fa5b4cb75e3c60d496760921b91df5e5e631e8e9e50c9d80350249c'.decode('hex')

for i in range(4):
    k=str2num(plain[i*16:(i+1)*16])^str2num(cipher[i*16:(i+1)*16])
    if i==0:
        print 'k0'
        print k
    #print 'k%d'%i
    #print parse(k)

print '---'
invk0='x^125 + x^124 + x^123 + x^121 + x^120 + x^118 + x^116 + x^114 + x^110 + x^109 + x^104 + x^103 + x^101 + x^96 + x^95 + x^94 + x^93 + x^88 + x^87 + x^86 + x^85 + x^82 + x^81 + x^80 + x^79 + x^77 + x^74 + x^72 + x^71 + x^66 + x^62 + x^51 + x^50 + x^44 + x^42 + x^41 + x^40 + x^38 + x^36 + x^35 + x^34 + x^33 + x^32 + x^29 + x^27 + x^25 + x^24 + x^23 + x^21 + x^20 + x^19 + x^17 + x^16 + x^15 + x^13 + x^11 + x^9 + x^5 + x^4 + x + 1'
k1andC='x^127 + x^126 + x^124 + x^123 + x^107 + x^105 + x^101 + x^99 + x^98 + x^95 + x^94 + x^93 + x^91 + x^90 + x^87 + x^86 + x^85 + x^83 + x^82 + x^81 + x^79 + x^77 + x^76 + x^72 + x^71 + x^66 + x^64 + x^63 + x^61 + x^59 + x^57 + x^54 + x^53 + x^52 + x^51 + x^49 + x^48 + x^47 + x^46 + x^45 + x^44 + x^41 + x^40 + x^38 + x^37 + x^36 + x^33 + x^32 + x^27 + x^25 + x^23 + x^22 + x^21 + x^20 + x^19 + x^18 + x^15 + x^13 + x^11 + x^10 + x^9 + x^7 + x^5 + x^3 + x^2 + 1'
print 'r1'
r1=mul(unparse(invk0),unparse(k1andC))
print parse(r1)
invk1='x^126 + x^125 + x^124 + x^123 + x^120 + x^117 + x^116 + x^115 + x^113 + x^112 + x^109 + x^107 + x^106 + x^105 + x^102 + x^101 + x^99 + x^98 + x^95 + x^92 + x^91 + x^89 + x^88 + x^87 + x^86 + x^82 + x^77 + x^76 + x^74 + x^73 + x^71 + x^64 + x^63 + x^62 + x^60 + x^59 + x^56 + x^53 + x^51 + x^50 + x^48 + x^46 + x^41 + x^40 + x^39 + x^38 + x^37 + x^36 + x^35 + x^34 + x^22 + x^20 + x^18 + x^14 + x^13 + x^8 + x^7 + x^6 + x^4 + x^3 + x^2 + x'
k2andC='x^126 + x^125 + x^124 + x^122 + x^118 + x^117 + x^116 + x^114 + x^113 + x^112 + x^111 + x^110 + x^107 + x^104 + x^103 + x^101 + x^100 + x^98 + x^96 + x^95 + x^94 + x^92 + x^90 + x^89 + x^88 + x^85 + x^84 + x^81 + x^79 + x^78 + x^75 + x^74 + x^73 + x^71 + x^69 + x^67 + x^64 + x^63 + x^62 + x^61 + x^60 + x^58 + x^56 + x^54 + x^53 + x^52 + x^50 + x^49 + x^48 + x^47 + x^46 + x^44 + x^41 + x^40 + x^38 + x^37 + x^36 + x^32 + x^27 + x^26 + x^23 + x^22 + x^21 + x^19 + x^17 + x^15 + x^14 + x^13 + x^12 + x^11 + x^9 + x^6 + x^4 + x^2'
print 'r2'
r2=mul(unparse(invk1),unparse(k2andC))
print parse(r2)
```


sage运行过程如下：
```
sage: F.<b> = GF(2)[]
sage: S.<x> = GF( 2**128, modulus = b^128 + b^7 + b^2 + b + 1 )
sage: A=x^127+x^126+x^122+x^121+x^119+x^117+x^114+x^112+x^110+x^109+x^108+x^106+x^105+x^104+x^102+x^101+x^100+x^99+x^98+x^97+x^96+x^94+x^91+x^90+x^88+x^87+x^86+x^82+x^81+x^77+x^76+x^75+x^72+x^71+x^70+x^68+x^66+x^65+x^64+x^63+x^62+x^60+x^56+x^55+x^53+x^50+x^48+x^43+x^42+x^40+x^38+x^37+x^34+x^32+x^29+x^24+x^23+x^22+x^21+x^18+x^17+x^16+x^15+x^12+x^11+x^9+x^8+x^7+x^6+x^5+x^4+x^3+x^2+1
sage: B=x^125+x^123+x^122+x^121+x^116+x^115+x^110+x^109+x^108+x^104+x^102+x^101+x^98+x^94+x^88+x^87+x^86+x^84+x^83+x^81+x^80+x^77+x^74+x^71+x^69+x^68+x^67+x^65+x^63+x^62+x^61+x^60+x^58+x^57+x^56+x^55+x^52+x^51+x^50+x^49+x^48+x^47+x^46+x^45+x^44+x^43+x^40+x^37+x^33+x^32+x^31+x^28+x^25+x^24+x^22+x^21+x^20+x^17+x^16+x^14+x^12+x^9+x^8+x^6+x^2+1
sage: inv1=(A+1)^(-1)
sage: inv1
x^127 + x^126 + x^125 + x^124 + x^122 + x^119 + x^113 + x^112 + x^109 + x^107 + x^103 + x^102 + x^101 + x^100 + x^99 + x^98 + x^96 + x^92 + x^91 + x^90 + x^87 + x^85 + x^84 + x^78 + x^75 + x^72 + x^59 + x^58 + x^56 + x^54 + x^53 + x^50 + x^46 + x^45 + x^44 + x^42 + x^40 + x^38 + x^37 + x^36 + x^33 + x^31 + x^30 + x^29 + x^28 + x^27 + x^25 + x^23 + x^22 + x^14 + x^13 + x^11 + x^9 + x^8 + x^4 + x^3
sage: Const=x^126+x^125+x^123+x^120+x^117+x^116+x^114+x^111+x^109+x^107+x^106+x^103+x^98+x^97+x^96+x^95+x^94+x^92+x^91+x^90+x^89+x^86+x^80+x^79+x^78+x^73+x^71+x^70+x^68+x^66+x^65+x^64+x^63+x^61+x^57+x^55+x^53+x^52+x^51+x^49+x^48+x^46+x^40+x^39+x^36+x^33+x^32+x^31+x^29+x^27+x^25+x^24+x^23+x^22+x^20+x^15+x^12+x^9+x^8+x^4+x^3+x
sage: k0=x^126+x^121+x^119+x^118+x^114+x^113+x^111+x^107+x^106+x^103+x^102+x^98+x^96+x^92+x^91+x^90+x^89+x^87+x^86+x^85+x^84+x^79+x^77+x^76+x^75+x^74+x^73+x^72+x^69+x^67+x^61+x^59+x^58+x^57+x^56+x^53+x^51+x^47+x^46+x^45+x^43+x^42+x^40+x^36+x^34+x^32+x^30+x^27+x^26+x^25+x^23+x^20+x^15+x^12+x^11+x^9+x^7+x^5+x^4+x^3+x^2
sage: k1=x^127+x^125+x^124+x^120+x^117+x^116+x^114+x^111+x^109+x^106+x^105+x^103+x^101+x^99+x^97+x^96+x^93+x^92+x^89+x^87+x^85+x^83+x^82+x^81+x^80+x^78+x^77+x^76+x^73+x^72+x^70+x^68+x^65+x^59+x^55+x^54+x^47+x^45+x^44+x^41+x^39+x^38+x^37+x^31+x^29+x^24+x^21+x^19+x^18+x^13+x^12+x^11+x^10+x^8+x^7+x^5+x^4+x^2+x+1
sage: k2=x^124+x^123+x^122+x^120+x^118+x^113+x^112+x^110+x^109+x^106+x^104+x^101+x^100+x^97+x^91+x^88+x^86+x^85+x^84+x^81+x^80+x^75+x^74+x^70+x^69+x^68+x^67+x^66+x^65+x^62+x^60+x^58+x^57+x^56+x^55+x^54+x^51+x^50+x^47+x^44+x^41+x^39+x^38+x^37+x^33+x^31+x^29+x^26+x^25+x^24+x^21+x^20+x^19+x^17+x^14+x^13+x^11+x^8+x^6+x^3+x^2+x
sage: k3=x^125+x^123+x^120+x^119+x^115+x^114+x^113+x^112+x^108+x^107+x^105+x^100+x^98+x^97+x^95+x^93+x^92+x^88+x^84+x^83+x^82+x^81+x^79+x^74+x^72+x^71+x^68+x^64+x^63+x^61+x^60+x^59+x^54+x^53+x^50+x^48+x^47+x^43+x^42+x^37+x^35+x^34+x^33+x^31+x^28+x^27+x^26+x^23+x^22+x^21+x^19+x^18+x^15+x^14+x^12+x^11+x^6+x^4
sage: invk0=(k0+Const)^(-1)
sage: invk0
x^125 + x^124 + x^123 + x^121 + x^120 + x^118 + x^116 + x^114 + x^110 + x^109 + x^104 + x^103 + x^101 + x^96 + x^95 + x^94 + x^93 + x^88 + x^87 + x^86 + x^85 + x^82 + x^81 + x^80 + x^79 + x^77 + x^74 + x^72 + x^71 + x^66 + x^62 + x^51 + x^50 + x^44 + x^42 + x^41 + x^40 + x^38 + x^36 + x^35 + x^34 + x^33 + x^32 + x^29 + x^27 + x^25 + x^24 + x^23 + x^21 + x^20 + x^19 + x^17 + x^16 + x^15 + x^13 + x^11 + x^9 + x^5 + x^4 + x + 1
sage: (k1+Const)
x^127 + x^126 + x^124 + x^123 + x^107 + x^105 + x^101 + x^99 + x^98 + x^95 + x^94 + x^93 + x^91 + x^90 + x^87 + x^86 + x^85 + x^83 + x^82 + x^81 + x^79 + x^77 + x^76 + x^72 + x^71 + x^66 + x^64 + x^63 + x^61 + x^59 + x^57 + x^54 + x^53 + x^52 + x^51 + x^49 + x^48 + x^47 + x^46 + x^45 + x^44 + x^41 + x^40 + x^38 + x^37 + x^36 + x^33 + x^32 + x^27 + x^25 + x^23 + x^22 + x^21 + x^20 + x^19 + x^18 + x^15 + x^13 + x^11 + x^10 + x^9 + x^7 + x^5 + x^3 + x^2 + 1
sage: r1=x^123+x^117+x^116+x^114+x^113+x^112+x^110+x^109+x^108+x^107+x^104+x^99+x^98+x^97+x^95+x^93+x^91+x^89+x^87+x^84+x^83+x^82+x^81+x^80+x^78+x^74+x^69+x^68+x^66+x^62+x^57+x^53+x^52+x^51+x^45+x^43+x^42+x^41+x^38+x^37+x^35+x^33+x^32+x^31+x^30+x^29+x^28+x^25+x^22+x^19+x^18+x^17+x^15+x^14+x^13+x^12+x^10+x^8+x^5+x^2+1
sage: invk1=(k1+Const)^(-1)
sage: invk1
x^126 + x^125 + x^124 + x^123 + x^120 + x^117 + x^116 + x^115 + x^113 + x^112 + x^109 + x^107 + x^106 + x^105 + x^102 + x^101 + x^99 + x^98 + x^95 + x^92 + x^91 + x^89 + x^88 + x^87 + x^86 + x^82 + x^77 + x^76 + x^74 + x^73 + x^71 + x^64 + x^63 + x^62 + x^60 + x^59 + x^56 + x^53 + x^51 + x^50 + x^48 + x^46 + x^41 + x^40 + x^39 + x^38 + x^37 + x^36 + x^35 + x^34 + x^22 + x^20 + x^18 + x^14 + x^13 + x^8 + x^7 + x^6 + x^4 + x^3 + x^2 + x
sage: k2+Const
x^126 + x^125 + x^124 + x^122 + x^118 + x^117 + x^116 + x^114 + x^113 + x^112 + x^111 + x^110 + x^107 + x^104 + x^103 + x^101 + x^100 + x^98 + x^96 + x^95 + x^94 + x^92 + x^90 + x^89 + x^88 + x^85 + x^84 + x^81 + x^79 + x^78 + x^75 + x^74 + x^73 + x^71 + x^69 + x^67 + x^64 + x^63 + x^62 + x^61 + x^60 + x^58 + x^56 + x^54 + x^53 + x^52 + x^50 + x^49 + x^48 + x^47 + x^46 + x^44 + x^41 + x^40 + x^38 + x^37 + x^36 + x^32 + x^27 + x^26 + x^23 + x^22 + x^21 + x^19 + x^17 + x^15 + x^14 + x^13 + x^12 + x^11 + x^9 + x^6 + x^4 + x^2
sage: r2=x^121+x^118+x^117+x^111+x^110+x^109+x^104+x^101+x^98+x^97+x^94+x^91+x^86+x^85+x^84+x^82+x^81+x^80+x^79+x^78+x^76+x^74+x^73+x^70+x^69+x^66+x^65+x^63+x^57+x^56+x^52+x^47+x^43+x^41+x^39+x^38+x^37+x^36+x^35+x^30+x^29+x^26+x^24+x^19+x^16+x^15+x^14+x^13+x^11+x^8+x^7+x^3+x^2+x+1
sage: A
x^127 + x^126 + x^122 + x^121 + x^119 + x^117 + x^114 + x^112 + x^110 + x^109 + x^108 + x^106 + x^105 + x^104 + x^102 + x^101 + x^100 + x^99 + x^98 + x^97 + x^96 + x^94 + x^91 + x^90 + x^88 + x^87 + x^86 + x^82 + x^81 + x^77 + x^76 + x^75 + x^72 + x^71 + x^70 + x^68 + x^66 + x^65 + x^64 + x^63 + x^62 + x^60 + x^56 + x^55 + x^53 + x^50 + x^48 + x^43 + x^42 + x^40 + x^38 + x^37 + x^34 + x^32 + x^29 + x^24 + x^23 + x^22 + x^21 + x^18 + x^17 + x^16 + x^15 + x^12 + x^11 + x^9 + x^8 + x^7 + x^6 + x^5 + x^4 + x^3 + x^2 + 1
sage: r1
x^123 + x^117 + x^116 + x^114 + x^113 + x^112 + x^110 + x^109 + x^108 + x^107 + x^104 + x^99 + x^98 + x^97 + x^95 + x^93 + x^91 + x^89 + x^87 + x^84 + x^83 + x^82 + x^81 + x^80 + x^78 + x^74 + x^69 + x^68 + x^66 + x^62 + x^57 + x^53 + x^52 + x^51 + x^45 + x^43 + x^42 + x^41 + x^38 + x^37 + x^35 + x^33 + x^32 + x^31 + x^30 + x^29 + x^28 + x^25 + x^22 + x^19 + x^18 + x^17 + x^15 + x^14 + x^13 + x^12 + x^10 + x^8 + x^5 + x^2 + 1
sage: discrete_log(r1,A,S.order()-1)
76716889654539547639031458229653027958
sage:
```

根据N得到flag为`flag{LCG1sN3ver5aFe!!}`
