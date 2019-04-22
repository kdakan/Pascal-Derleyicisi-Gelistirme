# PASCAL DERLEYİCİSİ GELİŞTİRME

1997 senesinde İTÜ Matematik Mühendisliği son sınıftayken yapmış olduğum bitirme tezim "Pascal derleyicisi geliştirme" ile ilgili elimde kalan pek birşey yok. O tarihte 10-15 bin satır C kodu yazmış olduğumu hatırlıyorum, 3 ay full-time uğraşmıştım. O tarihdeki floppy disket de 2000 lerin başlarında ölmüştü. Ancak 1998'de fortunecity.com da bu derleyiciyi tanıttığım bir giriş yazım vardı, o yazının kopyasını da https://web.archive.org adresinden bulabildim. Burada derlemiş olduğum bilgileri tekrar formatlayıp github'a koydum. O dönem kullandığım teknikleri gayet açık anlatmışım. Şimdi sıfırdan geliştirecek olsam da aynı teknikleri kullanırdım, ancak yacc/lex/c yerine muhtemelen antlr/c# ile geliştirirdim.

- [ 1. Derleyici bileşenleri](#1-derleyici-bileşenleri)
- [ 2. Gramer, Üretim, Alfabe, Dil](#2-gramer-üretim-alfabe-dil)
- [ 3. Gramerlerin Sınıflandırılması](#3-gramerlerin-sınıflandırılması)
- [ 4. Gramerlerin Denkliği, Belirsizliği](#4-gramerlerin-denkliği-belirsizliği)
- [ 5. Nitelikler (attributes) ve Genişletilmiş Gramer](#5-nitelikler-attributes-ve-genişletilmiş-gramer)
- [ 6. Lex ve YACC kullanımı](#6-lex-ve-yacc-kullanımı)
- [ 7. Lex dosya yapısı](#7-lex-dosya-yapısı)
- [ 8. Lex düzgün-ifadeleri ve operatörler](#8-lex-düzgün-ifadeleri-ve-operatörler)
- [ 9. Lex tanımları](#9-lex-tanımları)
- [10. YACC dosya yapısı](#10-yacc-dosya-yapısı)
- [11. YACC Tanımları](#11-yacc-tanımları)
- [12. Ağaç Türetim Örneği](#12-ağaç-türetim-örneği)
- [13. Değişkenlerin depolanışı ve Fonksiyonların bağlantısı](#13-değişkenlerin-depolanışı-ve-fonksiyonların-bağlantısı)
- [14. Sembol tablosu](#14-sembol-tablosu)
- [15. Genel olarak P-kodlar](#15-genel-olarak-p-kodlar)
- [16. P-kod detayları](#16-p-kod-detayları)
- [17. Temel Pascal yapıları için kod şablonları](#17-temel-pascal-yapıları-için-kod-şablonları)
- [18. Derleyicide kullanılan veri yapıları](#18-derleyicide-kullanılan-veri-yapıları)
- [19. Derleyicide kullanılan tipler ve fonksiyonlar](#19-derleyicide-kullanılan-tipler-ve-fonksiyonlar)

## 1. Derleyici bileşenleri:
Bir kaynak programın derlenmesi, şu aşamalardan oluşur:

1. Tarayıcı (scanner/lexer) (kelime analizi yapar, kelime tablosunu üretir)
2. Ayrıştırıcı (parser) (sözdizimi analizi & anlam analizi yapar, sembol tablosunu üretir ve kontrol amaçlı kullanır)
3. Kod üretici (code generator) (kod üretimi yapar, makine kodu veya arakod üretir)
4. Optimizasyon (kod optimizasyonu, yani en iyileştirmesi yapar, hız ve bellek kullanımı açısından optimize edilmiş makine kodu üretir)

- Kelime analizinin amacı, dili oluşturan kelimeleri (token) tanımak ve bunları sözdizimi analizi için parser'a iletmektir. 
- Ayrıştırıcı (parser), tarayıcıdan ardarda aldığı kelimelere, dilin özel (ayrılmış) kelime ve işaretleri yardımıyla sözdizimsel anlamlar yükler. Bu, bir cümledeki kelimelere özne, nesne, zarf, yüklem gibi görevleri yüklemeye benzer. 
- Bu kelimeler tanındıkça sembol tablosu denen veri yapısına birtakım yan bilgilerle birlikte (değişken tipleri, adresleri, büyüklükleri, vb.) birlikte kaydedilir. Aynı kelimeler tekrar kullanıldıklarında, kullanıldıkları yere göre sembol tablosunda kayıtlı bulunan bilgiler alınarak hata kontrolü yapılır, hata var ise uygun bir sözdimi veya semantik hata kodu üretilir. 
- Son aşama ise, bu kelimelerin cümledeki rolü ve anlamını kullanarak,buna uygun makine kodları üretmektir. Gerekirse üretilen kod optimize edilerek hızlandırılabilir, veya bellekte tuttuğu alan küçültülebilir. 
- Optimizasyonun daha etkin yapılabilmesi için ve değişik makineler için kod üretilebilmesi için, bu işler için daha uygun ve daha genel bir arakod üretilip, daha sonra bu arakod optimize edilip istenen makine için kod üretilebilir. 
- Bazı derleyiciler ise sanal bir makine için hızlı yorumlanabilen bir kod üretir ve bu kodu yorumlayarak programı değişik makinelerde çalıştırır.

## 2. Gramer, Üretim, Alfabe, Dil: 
Bir dilin grameri (G), aşağıda gösterilen sade örnekteki gibi:

- cümle -> özne nesne yüklem
- özne -> Ben | isim
- nesne -> kitabı | eve
- yüklem -> gittim | aldı

şablon üretim kurallarıyla belirlenir. 

- Burada | sembolü veya anlamındadır. Okun sol tarafındaki sembollerden sağ taraftaki sembollerin türetileceğini anlatan bu kuralların herbirine üretim denir. Üretim kurallarının kümesi P ile gösterilir. 
- Bir gramerin üretimlerinin hiçbirinde sol tarafta yer almamış olan sembollere uç (terminal) semboller denir. Yukarıdaki örnekte bunlar Ben, isim, kitabı, eve, gittim ve aldı sembolleridir. 
- Uç sembollerin kümesi T ile gösterilir. Örneğin, "Mehmet kitabı aldı" cümlesi tarayıcı tarafından parser'a isim(Mehmet), kitabı,aldı şeklinde iletilir. Parantez içindeki Mehmet, isim uç sembolünün taşıdığı niteliktir. 
- Uç sembollerle birlikte parser'a iletilen bu nitelikler, parser tarafından üretimler uygulanırken işlenir. 
- Bu örnekte parser, ileride kullanmak için Mehmet'in isim olduğu bilgisini sembol tablosuna kaydeder. Daha sonra üretimleri sembollere uygulayarak cümle sembolünü elde eder. 
- Uç semboller dışındaki sembollere ara (non-terminal) semboller denir. Bu örnekte bunlar cümle, özne, nesne ve yüklem'dir. Ara sembollerin kümesi N ile gösterilir. 
- Bütün sembolleri kendisinden türeten özel ara sembole cümlesel sembol denir ve S ile gösterilir. Bu örnekte cümlesel sembol cümle'dir. 
- S, P, N, T dörtlüsüne gramer denir ve G={S,P,N,T} ile gösterilir. 
- V=N∪T (N, T birleşim) kümesine alfabe denir. 
- S sembolüne ardarda uygulanan P'ye ait üretimler sonucu elde edilen ve uç sembollerden oluşan sembol grubuna cümle denir. 
- Bir gramerden türetilebilecek tüm cümlelerin kümesine ise dil denir ve L(G) ile gösterilir. 
- EPSİLON ile gösterilen özel uç sembol, boşluğa, daha doğrusu boşluk karakterine değil de, hiçbirşeye karşılık gelir.

## 3. Gramerlerin Sınıflandırılması:
Chomsky sınıflandırmasına göre gramerler genelden özele doğru birbirlerini içeren 4 sınıfa ayrılırlar:

1. Serbest gramerler:
Üretimler u -> v şeklindedir. Burada u,v∈V ve u≠EPSİLON dur.

2. Bağlam-duyarlı (context-sensitive) gramerler:
Üretimler uxv -> uvw şeklindedir. Burada u,v,w⊂V, v≠EPSİLON ve x∈N dir. Yani, x ara sembolü, yalnızca u ve w sembol grupları ile çevrelendiği takdirde v sembol grubu ile türetilebilir.

3. Bağlamdan-bağımsız (context-free) gramerler:
Üretimler x->v şeklindedir. Burada v∈V ve xeN dir. Programlama dilleri genellikle bu tür gramerlerden üretilir. Bu tür diller her zaman tanınabilir.

4. Sonlu (veya düzenli) gramerler:
Üretimler x->a veya x->ay şeklindedir. Burada x,y∈N ve a∈T dir. Bu tür gramerler, dilin kelimelerini tanımlamak için kullanılır, yani kelime analizi sırasında ayrıştırılan bir gramerdir. Sonlu gramerler, sonlu durum makineleriyle çok etkin şekilde ayrıştırılabilir.

Kelime ve sözdizimi analizinin ayrı ayrı yapılmasının nedeni de, ayrıştırılan dillerin farklı gramer sınıflarından olmasıdır. Kelime analizi çok daha etkin algoritmalarla yapılır.

## 4. Gramerlerin Denkliği, Belirsizliği:
L(G) ve L(H) kümeleri birbirine eşitse, G ve H gramerlerine denk gramerler denir. Üretimler farklı sıralarda uygulanıp aynı bir cümle farklı şekilde türetilebiliyorsa bu gramere belirsiz (ambiguous) gramer denir. Ayrıştırıcı (parser)'ın bu tür belirsiz bir grameri tek şekilde tanıması sağlanmalıdır, aksi halde alınacak sonuçlar her seferinde değişik olacaktır. Örneğin şu gramerde:

s -> aa
a -> x | xx

"xxx" cümlesi iki farklı sırada tanınabilir:

<table>
    <tbody><tr>
        <td align="center"></td>
        <td align="center"></td>
        <td align="center"><strong>s</strong></td>
        <td align="center"></td>
        <td align="center"></td>
        <td align="center"></td>
        <td align="center"></td>
        <td align="center"></td>
        <td align="center"><strong>s</strong></td>
        <td align="center"></td>
        <td align="center"></td>
    </tr>
    <tr>
        <td align="center"></td>
        <td align="right">/</td>
        <td align="center"></td>
        <td align="left">\</td>
        <td align="center"></td>
        <td align="center"><strong>veya</strong></td>
        <td align="center"></td>
        <td align="right">/</td>
        <td align="center"></td>
        <td align="left">\</td>
        <td align="center"></td>
    </tr>
    <tr>
        <td align="center"></td>
        <td align="center"><strong>a</strong></td>
        <td align="center"></td>
        <td align="center"><strong>a</strong></td>
        <td align="center"></td>
        <td rowspan="3"></td>
        <td align="center"></td>
        <td align="center"><strong>a</strong></td>
        <td align="center"></td>
        <td align="center"><strong>a</strong></td>
        <td align="center"></td>
    </tr>
    <tr>
        <td align="right">/</td>
        <td align="center"></td>
        <td align="right">/</td>
        <td align="center"></td>
        <td align="left">\</td>
        <td align="right">/</td>
        <td align="center"></td>
        <td align="left">\</td>
        <td align="center"></td>
        <td align="left">\</td>
    </tr>
    <tr>
        <td align="center"><strong>x</strong></td>
        <td align="center"></td>
        <td align="center"><strong>x</strong></td>
        <td align="center"></td>
        <td align="center"><strong>x</strong></td>
        <td align="center"><strong>x</strong></td>
        <td align="center"></td>
        <td align="center"><strong>x</strong></td>
        <td align="center"></td>
        <td align="center"><strong>x</strong></td>
    </tr>
</tbody></table>

Bunlardan birisi parser tarafından tercih edilmelidir.

## 5. Nitelikler (attributes) ve Genişletilmiş Gramer:
Her sembolün taşıdığı kendisine ait nitelikleri bulunur. Üretimlerin yanında bu niteliklerle ilgili işlemler konarak elde edilen gramere genişletilmiş gramer denir. Sol taraftaki sembolün niteliği, sağda yeralan sembollerin nitelikleri işlenerek oluşuyorsa, yani bunların bir fonksiyonuysa:

x -> y1 y2 y3 ... yn ve x.a = f(y1.a, y2.a, y3.a,...,yn.a) ise

(burada x.a ile x'in taşıdığı a niteliği gösteriliyor), bu niteliklere sentezlenmiş nitelikler (synthesized attributes) denir.

## 6. Lex ve YACC kullanımı:
UNIX sisteminde mevcut olan Lex ve YACC programları, derleyici oluşturmada çok büyük kolaylıklar sağlar. Lex programı, verilen bir sonlu grameri kabul eden, yani tanıyan tarayıcı üretir. YACC de benzer şekilde, verilen genişletilmiş bir LALR grameri kabul eden ve üretimler sırasında, bunların yanında verilmiş olan sıfatlarla ilgili işlemleri yapan bir parser üretir. İki programın çıktısı da, yani oluşturdukları tarayıcı ve parser programlar C dili ile yazılmıştır.

## 7. Lex dosya yapısı:
```
%{ 
C ifadeleri (seçimlik)
%} 
lex tanımları (seçimlik)
%% 
lex düzgün-ifadeleri ve işlemler
%% (seçimlik)
C ifadeleri (kullanıcı fonksiyonları)
```

## 8. Lex düzgün-ifadeleri ve operatörler:
Lex'de kullanılan düzgün-ifadeler, lex operatörleri ve tanınması istenen karakterlerden oluşur.
```
.	\n harici tek bir karakter	.a	satır başında olmayan a ve a'dan önceki karakter
*	sıfır veya daha fazla adet	a[a-z]*	a ile başlayan küçük harfli kelimeler (a dahil)
+	bir veya daha fazla adet	a[a-z]+	a ile başlayan küçük harfli kelimeler (a hariç)
?	seçimlik (var veya yok)	        XY?Z	XZ veya XYZ
|	veya (ya sol ya sağ taraf)	a|b	a veya b
xy	bitiştirme (x ve y yanyana)	abc	ardarda a, b ve c (abc)
( )	gruplandırma	                (UNIX)|(Unix)	UNIX veya Unix
" "	operatörlerin anlamını iptal	c"++"	c++ kelimesi
\	1. tek op.ün anlamını iptal	c\+\+	c++ kelimesi
        2. C escape karakterleri	\n	satır sonu karakteri
        3. sekizlik düzende karşılığı	\176	~ karakteri (ASCII=176)
[ ]	karakter sınıfı	                [A-Z]	bir büyük harf
[^ ]	karakter sınıfını hariç tut	[^XYZ]	X,Y ve Z dışında bir karakter
^	satır başında ifade	        ^(abc)	satır başında abc
$	satır sonunda ifade	        $a	satır sonunda a
{m,n}	en az m, en çok n adet ifade	a{1,5}	1 ila 5 adet a
```

## 9. Lex tanımları:
kelime düzgün-ifade çiftlerinden oluşur. Soldaki kelime, karşılığı olan sağdaki ifade yerine diğer ifadelerde kullanılabilir.

Örnek:
```
tamsayı [0-9]+
%%
tamsayı          {sscanf(yytext, "%d", &yylval.ival); return TAMS;}
tamsayı\.tamsayı {sscanf(yytext, "%f", &yylval.fval); return REELS;}
```
Örnekte görüldüğü gibi işlemler `{ }` içinde yeralan C ifadeleridir. `yytext` değişkeni karakter dizisi olup, tanınan kelimenin (örnekte 15, 6349 gibi bir tamsayı veya 0.359, 1368.4 gibi bir reel sayı) saklandığı alandır. Lex tarafından oluşturulan ve parser tarafından çağırılan `yylex()` tarayıcı fonksiyonu tanıdığı tokeni dönerek parser'a iletir. Bunun dışında tokenin niteliğini de `yylval` global değişkeni ile iletir. `{ }` içindeki ifadeler, C bloku içinde yeraldığından, burada lokal C değişkeni tanımlamak da mümkündür. `yylval` değişkeninin tipi, YACC'nin oluşturduğu parser içinde tanımlanmıştır.

## 10. YACC dosya yapısı:
```
%{ -I
C ifadeleri I-> seçimlik
%} -I
YACC tanımları
%%
YACC üretim kuralları ve işlemler
%%                 
C ifadeleri (kullanıcı fonksiyonları ve main() fonksiyonu)
```

## 11. YACC tanımları:
`%union{ }`: `yylval` değişkeninin tipini C'deki `union` tipi yapar. Bu istenmiyorsa, `%{ %}` içine `#define YYSTYPE tip` şeklinde bir ifade konmalıdır. `%union{ }` içinde `union`'a ait sahalar yeralmalıdır.

`%token`: uç semboller,` %token sembol` şeklinde tanımlanmalıdır. Eğer `%union{ }` kullanıldıysa, tokenin tipi, `union` içinde o tipteki sahanın adı t olmak üzere `%token <t>` sembol şeklinde belirtilmelidir.

`%left`, `%right`: işlemin sola veya sağa birleşim niteliği varsa `%left işlem` veya `%right işlem` şeklinde belirtilir. Bunlar, gramerdeki belirsizliği gidermek için kullanılır.

`%nonassoc`: işlemin sağa veya sola birleşme niteliği olmadığını belirtir.

`%type`: uç semboller için `%union{ }` kullanıldıysa yapılan tip tanımının ara semboller için benzeridir.

İşlemlerin öncelikleri `%left`, `%right`, veya `%nonassoc` ile tanımlandıkları satırın konumuyla orantılıdır. Tanımı aşağıda yeralan işlemin önceliği, daha yukarıda tanımlanmış olan işlemin önceliğinden üstündür. Tanımlarda alt satırlara inildikçe işlemlerin önceliği artar. 

## 12. Ağaç Türetim Örneği:
```
%{
#define  YYSTYPE int /* yylval tipi ve YACC stack tipi tamsayı */
%}
%token   TAMSAYI
%left    '+' '-'
%left    '*' '/'
%right   '^'
%left EKSİ
%%
ifade    : ifade '+' ifade       {$$=$1+$3;}
         | ifade '-' ifade       {$$=$1-$3;}
         | ifade '*' ifade       {$$=$1*$3;}
         | ifade '/' ifade       {$$=$1/$3;}
         | ifade '^' ifade       {$$=power($1,$3);}
         | '-' ifade %prec EKSİ  {$$=-$2;}
         | TAMSAYI               {$$=$1;}
         ;
%%
...
```
gramerinde `1+2*3*4^5` cümlesi aşağıdan yukarı doğru şu sırayla türetilir:

<table>
  <tbody>
    <tr>
        <td align="center"></td>
        <td align="center"></td>
        <td align="center"></td>
        <td align="center"></td>
        <td align="center">+</td>
        <td align="center"></td>
        <td align="center"></td>
        <td align="center"></td>
        <td align="center"></td>
    </tr>  
    <tr>
        <td align="center"></td>
        <td align="center"></td>
        <td align="center"></td>
        <td align="center">/</td>
        <td align="center"></td>
        <td align="center">\</td>
        <td align="center"></td>
        <td align="center"></td>
        <td align="center"></td>
    </tr>
    <tr>
        <td align="center"></td>
        <td align="center"></td>
        <td align="center">/</td>
        <td align="center"></td>
        <td align="center"></td>
        <td align="center">*</td>
        <td align="center"></td>
        <td align="center"></td>
        <td align="center"></td>
    </tr>
    <tr>
        <td align="center"></td>
        <td align="center">/</td>
        <td align="center"></td>
        <td align="center"></td>
        <td align="center">/</td>
        <td align="center"></td>
        <td align="center">\</td>
        <td align="center"></td>
        <td align="center"></td>
    </tr>
    <tr>
        <td align="center">/</td>
        <td align="center"></td>
        <td align="center"></td>
        <td align="center">*</td>
        <td align="center"></td>
        <td align="center"></td>
        <td align="center"></td>
        <td align="center">^</td>
        <td align="center"></td>
    </tr>
    <tr>
        <td align="center">|</td>
        <td align="center"></td>
        <td align="center">/</td>
        <td align="center"></td>
        <td align="center">\</td>
        <td align="center"></td>
        <td align="center">/</td>
        <td align="center"></td>
        <td align="center">\</td>
    </tr>
    <tr>
        <td align="center">1</td>
        <td align="center"></td>
        <td align="center">2</td>
        <td align="center"></td>
        <td align="center">3</td>
        <td align="center"></td>
        <td align="center">4</td>
        <td align="center"></td>
        <td align="center">5</td>
    </tr>
  </tbody>
</table>

- Türetme, aynen bu ağaç yapısının postorder taranma sırasını izler. Yani her dal için önce o dalın soldan başlayarak tüm alt dalları taranır, sonra da dalın kendisi taranır. 
- Üretim kurallarının yanında yeralan işlemler, { } içerisinde C ifadeleridir (C bloku). Burada $n şeklindeki hayali değişkenler, sembollerin sıfatlarına karşılık gelirler. Soldaki ara sembolün sıfatı $0 veya $$ ile, sağdaki k.ıncı sembolün sıfatı da $k ile gösterilir. { } içerisinde, diğer işlemlerin yanısıra $1,$2,...,$n işlenerek elde edilen sonuç $$'a atanır. 
- Bu değerler, aslında parser'ın kullandığı stack üzerinde sembollerle birlikte tutularak yukarı doğru işlenerek yolalır, yoksa gerçekte ağaç veri yapısı kullanılmaz. 
- En üst kuralın solundaki sembol, ya da %start ile tanımlanmış başka bir sembol, cümlesel semboldür. Tüm semboller, uç sembollerden başlamak üzere, sağ tarafında yeraldıkları kuralın solundaki ara sembole, aşağıdan yukarı doğru türetilirler. 
- Bu aşamalarda taşınan nitelikler kullanılarak ve işlenerek yukarı aşamalara geçirilerek, sonunda cümlesel sembole ulaşıldığında derlenme süreci biter. 
- Özel bir ara sembol olan error sembolü, derlenen programda karşılaşılan sözdizimi hatasının, sanki olmamış gibi error sembolüne türetilmesi ve hatalı kısmın atlanarak derlenmeye devam edilmesi için kullanılır.

## 13. Değişkenlerin depolanışı ve Fonksiyonların bağlantısı:
- Pascal dili blok yapılı bir dil olduğundan, C, BASIC ve FORTRAN'dan farklı olarak iç içe bloklar, yani iç içe fonksiyonlar tanımlanabilir. 
- Tanımlanan bir sembol, tanımlandığı blok ve bunun içindeki tüm alt bloklar içinden görülebilir, üst bloklardan görülemez. Depolanma yani yaşam süresi de, o bloğun ömrü kadardır. 
- Fonksiyon ve prosedürler, rekürsif biçimde kendi kendilerini çağırabilirler. Geçici (lokal) değişkenler de tanımlanabilir. Bütün bunlar, en uygun şekilde stack üzerinde sağlanabilir. 

Programın çalışma süresince stack'in aldığı genel yapı şöyledir:
```
değişken N <- T
...	 	 
değişken 1	 	 
dönüş adresi	 	 
dinamik link	 	 
statik link <- B
parametre M	 	 
...	 	 
parametre 1	 	 
dönüş değeri	 	 
```

- Burada B ve T, P-makinesi adı verilen ve derlenmiş kodun çalışacağı sanal makinenin iki yazmacıdır. T, stack'in tepesini gösteren, B ise stack'i adresleyen pointerdır. 
- Bütün adresler, B'nin tuttuğu base adrese pozitif veya negatif bir ofset eklenerek belirtilir. Parametre 1,...,M o an işleyen fonksiyonun parametreleri için, değişken1,...,N fonksiyonun içinde tanımlanmış olan lokal değişkenler için, dönüş değeri ise fonksiyonun dönüş değeri için ayrılmış alanı gösterir. 
- Statik link, fonksiyonu (bloku) içeren üst blok fonksiyonlarındaki değişkenlere, bu fonksiyon tarafınfan erişilebilmesi için konmuş olan, bir üst bloğun base adresidir. 
- Dinamik link, bu fonksiyonun işleyişi sona erdiğinde çağrıldığı noktaya geri döndüğünde, o fonksiyonun değişkenleri ve diğer stack yapısına tekrar dönmesini sağlayan, çağıran fonksiyonun base adresidir. 
- Dönüş adresi, fonksiyonun geri döndüğü zaman çalıştırılacak komutun adresidir. 
- P-makinesi, bütün komutları stack uzerinde işlem yapan bir sanal makinedir. Komutlarına P-kodu denir.

## 14. Sembol tablosu:
- Bir blokta tanımlanmış semboller yalnızca bu blok ve bu bloğun içindeki bloklar içerisinden görülebilir. Yukarıdaki stack yapısı sayesinde program işlerken bu sağlanır. 
- Derlenme aşamasında bunun sağlanması ve aynı isimli, farklı bloklarda yeralan sembollerin birbiriyle karıştırılmaması için derleyicide kullanılan sembol tablosu da stack yapısındadır. 

Örneğin, A bloğu içinde B ve C blokları yeralıyorsa sembol stack'inin durumu şöyledir:

A blok derlenirken:
```
A'nın sembolleri
```

B blok derlenirken:
```
B'nin sembolleri
A'nın sembolleri
```

C blok derlenirken:
```
C'nin sembolleri
A'nın sembolleri
```

## 15. Genel olarak P-kodlar:
```
p-kod      opkod   anlamı
LIT 0,N	    00	   N değerini stack'e yükle
OPR 0,N	    01	   stack üstünde aritmetik N işlemi yap
LOD L,N	    02	   değişkeni stack'e yükle
LODX L,N    12	   dizi değişkeni stack'e yükle
STO L,N	    03	   değişkeni stack'ten yükle
STOX L,N    13	   dizi değişkeni stack'ten yükle
CRL L,N	    04	   prosedür veya fonksiyon çağır
INT 0,N	    05	   T'ye poz. veya neg. sabit ekle
JMP 0,N	    06	   adrese dallan
JPC C,N	    07	   adrese koşullu dallan
CSP 0,N	    08	   standart prosedür çağır
```
Not: opkod onaltılık sayı düzenindedir.

## 16. P-kod detayları:
```
p-kod	    yaptığı iş	                              psüdokod
LIT 0,N	    N sabitini stack'e yükle                  PUSH N
OPR 0,0	    prosedür veya fonksiyondan geri dön       RETURN
OPR 0,1	    negatifini al                             POP A, PUSH (-A)
OPR 0,2	    toplamını al                              POP A, POP B, PUSH (B+A)
OPR 0,3	    farkını al                                POP A, POP B, PUSH (B-A)
OPR 0,4	    çarpımını al                              POP A, POP B, PUSH (B*A)
OPR 0,5	    bölümünü al                               POP A, POP B, PUSH (B/A)
OPR 0,6	    alt bitini al                             POP A, PUSH (A AND 1)
OPR 0,7	    kalanını al                               POP A, POP B, PUSH (B MOD A)
OPR 0,8	    eşit mi                                   POP A, POP B, PUSH (B =? A)
OPR 0,9	    farklı mı                                 POP A, POP B, PUSH (B ≠? A)
OPR 0,10    küçük mü                                  POP A, POP B, PUSH (B <? A)
OPR 0,11    büyük eşit mi                             POP A, POP B, PUSH (B >=? A)
OPR 0,12    büyük mü                                  POP A, POP B, PUSH (B >? A)
OPR 0,13    küçük eşit mi                             POP A, POP B, PUSH (B <=? A)
OPR 0,14    veya                                      POP A, POP B, PUSH (B OR A)
OPR 0,15    ve                                        POP A, POP B, PUSH (B AND A)
OPR 0,16    değil                                     POP A, PUSH (NOT A)
OPR 0,17    sola shift                                POP A, POP B, PUSH (B sola shift (A kere))
OPR 0,18    sağa shift                                POP A, POP B, PUSH (B sağa shift (A kere))
OPR 0,19    arttır                                    POP A, PUSH (A+1)
OPR 0,20    eksilt                                    POP A, PUSH (A-1)
OPR 0,21    kopyala                                   POP A, PUSH A, PUSH A
LOD L,D	    adresten yükle                            A'yi (L,D)'den yükle, PUSH A
LOD 255,0   stack üzerindeki adresten yükle           POP A, POP adres, A'nin alt byte'ini adrese yükle
LODX L,D    indexli yükle                             POP index, A'yi (L,D)+index'ten yükle, PUSH A
STO L,D	    stack üzerinden adrese yükle              POP A, A'yi (L,D)'ye yükle
STO 255,0   stack üzerinden stack'teki adrese yükle   POP A, POP adres, A'nin alt byte'ini adrese yükle
STOX L,D    indexli yükle                             POP index, POP A, A'yi (L,D)+index'e yükle
CAL L,A	    çağır                                     (L,A)'yı çağır
CAL 255,0   stack'teki adresi çağır                   POP adres, PUSH dönüş adresi (=pc), adrese dallan
INT 0,N	    T ye N ekle                               T=T+N
JMP 0,A	    dallan                                    (0,A)'ya dallan
JPC 0,A	    doğruysa dallan                           POP A, eğer (A and 1) = 0 ise (0,A)'ya dallan
CSP 0,0	    karakter girişi                           INPUT A, PUSH A
CSP 0,1	    karakter çıkışı                           POP A, OUTPUT A
CSP 0,2	    tamsayı girişi                            INPUT A#, PUSH A
CSP 0,3	    tamsayı çıkışı                            POP A, OUTPUT A#
CSP 0,8	    string çıkışı                             POP A, FOR I := 1 to A DO BEGIN POP B, OUTPUT B, END
```
Not: doğru=1, yanlış=0

POP X 'in anlamı: T'nin gösterdiği elemanı X'e yükle, T'yi 1 azalt.
PUSH X 'in anlamı: T'yi 1 arttır, X'i T'nin gösterdiği elemana yükle.

## 17. Temel Pascal yapıları için kod şablonları:
```
Pascal deyimi	               p-code karşılığı
x+10*y[5]                      LOD x
                               LIT 10
                               LIT 5
                               LODX Y
                               OPR *
                               OPR +
 	 
a:=expr;                       (expr)
                               STO a
 	 
p^=expr;                       (expr)
                               STO 255 p
 	 
if expr then stmt1 else stmt2; (expr)
                               JPC 0,1b1
                               (stmt1)
                               JMP 1b2
                               1b1: (stmt2)
                               1b2: ...
 	 
for i=expr1 to expr2 do stmt;  (expr1)
                               STO I
                               (expr2)
                               1b1: OPR CPY
                               LOD I
                               OPR >=
                               JPC 0,1b2
                               (stmt)
                               LOD I
                               OPR INC
                               STO I
                               JMP 1b1
                               1b2: INT -1
 	 
while expr do stmt             1b1: (expr)
                               JPC 0,1b2
                               (stmt)
                               JMP 1b1
                               1b2: ...
 	 
case expr of	               (expr)
c1b1,c1b2: stmt1;              OPR CPY
c1b3 : stmt2;	               LIT c1b1
else stmt3 end;	               OPR =
                               JPC 1,1b1
                               OPR CPY
                               LIT c1b2
                               OPR =
                               JPC 0,1b2
                               1b1: (stmt1)
                               JMP 1b4
                               1b2: OPR CPY
                               LIT c1b3
                               OPR =
                               JPC 0,1b3
                               (stmt2)
                               JMP 1b4
                               1b3: (stmt3)
                               1b4: INT -1
 	 
repeat stmt until expr;	       1b1: (stmt)
                               (expr)
                               JPC 0,1b1
 	 
i=funca(expr1,expr2);	       INT 1
                               (expr1)
                               (expr2)
                               CAL funca
                               INT -2
```

## 18. Derleyicide kullanılan veri yapıları:
- Bağlı listeler, dinamik diziler oluşturmak için kullanılır. 
- Listeyi oluşturacak her bir hucre için tek tek C'deki malloc() fonksiyonu ile dinamik bellek ayırılmalı ve bir sonraki hücreyle bağlantısı kurulmalıdır.

- Tek bağlı liste:
```
baş->d
     s->d
        s->d
           s->d
              s->0
```
Burada d'ler liste içinde sıralı şekilde tutulan bilgiler, s'ler de bir sonraki hücreyi gösteren pointer'lardır. İlk hücrenin adresi baş'da tutulur. En son hücrede s'nin değeri NULL, yani 0'dir.
C dilinde şu şekilde tanımlanır:
```
struct snode {
    int d;
    struct snode *s; /* sonraki */
} *baş;
```
- Çift bağlı liste:
```
     ö=0 
baş->d<-ö
     s->d<-ö 
        s->d<-ö
           s->d<-son
              s=0
```
Burada d'ler liste içinde sıralı şekilde tutulan bilgiler, s'ler bir sonraki hücreyi gosteren, ö'ler ise bir önceki hücreyi gösteren pointer'lardır. En baştaki hücrenin adresi baş'da, istenirse son hücrenin adresi son'da tutulur. En baş hücrede ö'nün değeri, en son hücrede s'nin değerleri NULL, yani 0'dir. 
C dilinde şu şekilde tanımlanır:
```
struct snode {
    int d;
    struct snode *ö; /* önceki */
    struct snode *s; /* sonraki */
} *baş, *son;
```
Dinamik Stack :

Çift bağlı liste, stack olarak kullanılırsa, son hücreyi gösteren
pointer da stack'in üstünü östermiş olur. Böylece büyüklüğü dinamik
olarak değişen bir stack yapısı elde edilir.

## 19. Derleyicide kullanılan tipler ve fonksiyonlar:

- `symblocktop`: Sembol tablosundaki (stack'teki) en üst elemanı gösteren pointer'dır.
- `pushsymblock()`: Sembol stack'ine yeni sembol bloku yükler. (stack'i 1 blok büyültür)
- `popsymblock()`: Sembol stack'inden bir sembol bloku atar. (stack'i 1 blok küçültür.
- `instconst()`, `instlabel()`, `insttype()`, `instvar()`: Sembol stack'inin üstündeki bloka sırasıyla yeni sabit, etiket, tip ve değişken ekler.
- `makeptrtype()`, `makeenumtype()`, `makerangetype()`, `makeidtype()`, `makerectype()`, `makearraytype()`, `makeuniontype()`,
   `makesettype()`, `makefiletype()`: Çeşitli tip hücreleri oluşturur. Değişkenlerin tipleri, sembol tablosunda birbirine bağlı tip hücreleri şeklinde tutulur.

Tip hücresi şöyle tanımlıdır:
```
struct stype {
    char metatype; /* TENUM, TID, TREC, TUNION, TFILE, TSET, TRANGE, TARR */
    void *restptr; 
};
```
Burada metatype `TENUM`, `TID`, `TREC`, `TUNION`, `TARR`, `TFILE`, `TSET` ve `TRANGE` sabitlerinden birini tutar. restptr, her tip için farklı türden bilgileri içeren bir yapıyı gösterir. Bu yapılar şöyledir:
<table>
    <tr>
        <td>metatype</td>
        <td>restptr</td>
    </tr>
    <tr>
        <td>TENUM</td>
        <td>-->id0-->id1-->...-->idN</td>
    </tr>
    <tr>
        <td></td>
        <td>(bağlı liste, id'ler string olup, ayrıca sembol tablosuna 0'dan N'ye kadar sabitler olarak kaydedilir)</td>
    </tr>
    <tr>
        <td>TRANGE</td>
        <td>-->altsınır,üstsınır,tip (tip karakter veya nümeriktir)</td>
    </tr>
    <tr>
        <td>Örnek:</td>
        <td>VAR r:(A..Z) altsınır=A, üstsınır=Z, tip=karakter</td>
    </tr>
    <tr>
        <td>TID</td>
        <td>-->id (id: eşdeğer tipin ismi)</td>
    </tr>
    <tr>
        <td>Örnek:</td>
        <td>TYPE sayıtipi = INTEGER; (INTEGER'in eşdegeri bir tip)
    </tr>
    <tr>
        <td></td>
        <td>VAR i:sayıtipi id="sayıtipi"</td></td>
    </tr>
    <tr>
        <td>TARRAY</td>
        <td>-->tip,inextipi1-->...-->indextipiN</td>
    </tr>
    <tr>
        <td></td>
        <td>(bağlı liste,N elemanlı dizideki indislerin tipleri)</td>
    </tr>
    <tr>
        <td>TREC ve TUNION</td>
        <td>-->fields1-->...-->fieldsN</td>
    </tr>
    <tr>
        <td></td>
        <td>(bağlı liste, her bir fields da, aynı tipten sahaların oluşturduğu bir bağlı liste ve tip çiftidir)</td>
    </tr>
    <tr>
        <td>Örnek:</td>
        <td>VAR u: UNION
f1,f2,f3: REAL; fields1->(f1,f2,f3,REAL)
i1,i2: INTEGER; fields2->(i1->i2,INTEGER)
END;</td>
    </tr>
    <tr>
        <td>TPTR</td>
        <td>-->tip (pointerın gösterdiği tip)</td>
    </tr>
    <tr>
        <td>Örnek:</td>
        <td>VAR p: ^ tip</td>
    </tr>
    <tr>
        <td>TFILE</td>
        <td>-->tip</td>
    </tr>
    <tr>
        <td>Örnek:</td>
        <td>VAR f: FILE OF CHAR; (tip=CHAR)</td>
    </tr>
    <tr>
        <td>TSET</td>
        <td>-->kümetipi</td>
    </tr>
    <tr>
        <td>Örnek:</td>
        <td>VAR s: SET OF INTEGER;</td>
    </tr>
</table>


`instproc()`, `instfunc()`: Sembol tablosunun üstündeki bloka prosedür ve fonksiyon ekler.

`addXnode()`: X'lerden oluşan bağlı listeye yeni hücre ekler.
Bu tür fonksiyonlar: `addidnode()`, `addtypenode()`, `addfieldsnode()`, `addfparsec()`, `addexprnode()`, `addclinenode()`, `addconstnode()`, `addvarnode()`

`linkNcodes()`: P-kodlardan oluşan N adet bağlı listeyi birleştirip, tek bir bağlı liste haline getirir.

`codeXXX()`: XXX P-kodunu üretir. Üretilen kod aslında veri kısmı P-kod olan tek elemanlı bir bağlı listedir. Kod üretimi sırasında değişik zamanlarda üretilen kod parçaları (bağlı listeler), `linkNcodes()` ile farklı aşamalarda birleştirilir ve en sonunda tek parça haline getirilir bu bağlı listenin adresi program isimli değişkene atanır.
```
