# PASCAL DERLEYİCİSİ GELİŞTİRME

1997 senesinde İTÜ Matematik Mühendisliği için yapmış olduğum bitirme tezim "Pascal derleyicisi geliştirilmesi" ile ilgili elimde kalan pek birşey yok. O tarihte 15000 satır C kodu yazmış olduğumu hatırlıyorum, 3 ay full time bunun için uğraşmıştım. O tarihdeki floppy disket de  2000 lerin başlarında ölmüştü. Ancak 1998'de fortunecity.com da bu derleyiciyi tanıttığım bir giriş yazım vardı, o yazının kopyasını da https://web.archive.org/web/20120308052921/http://members.fortunecity.com/kdakan/ adresinden bulabildim. Bu sitedeki arşiv görüntüsü okunaklı değil, sadece metin ve tablolar okunabiliyor, resimler arşivlenmemiş. Elimdeki bu son kırıntı bilgiyi sizler için tekrar formatlayıp github'a koymayı uygun gördüm, orjinal metin üzerinde hiçbir değişiklik yapmadım. O dönemde kullandığım teknikleri burada gayet açık anlatmışım. Şimdi sıfırdan yazacak olsam da aynı teknikleri kullanırdım, ancak yacc/lex/c yerine muhtemelen antlr/c# ile yazardım.

- [ 1. Derleyicinin bileşenleri](#1-derleyicinin-bileşenleri)
- [ 2. Gramer, Üretim, Alfabe, Dil](#2-gramer-üretim-alfabe-dil)
- [ 3. Gramerlerin Sınıflandırılması](#3-gramerlerin-sınıflandırılması)
- [ 4. Gramerlerin Denkliği, Belirsizliği](#4-gramerlerin-denkliği-belirsizliği)
- [ 5. Özellikler (attributes) ve Genişletilmiş Gramer(#5-özellikler-(attributes)-ve-genişletilmiş-gramer)
- [ 6. Lex ve YACC kullanımı](#6-lex-ve-YACC-kullanımı)
- [ 7. Lex'in kabul ettiği (.l uzantılı) dosya yapısı](#7-lex'in-kabul-ettiği-(.l-uzantılı)-dosya-yapısı)
- [ 8](#8-)
- [ 9](#9-)
- [10](#10-)
- [11](#11-)
- [12](#12-)
- [13](#13-)
- [14](#14-)
- [15](#15-)
- [16](#16-)


## 1. Derleyicinin bileşenleri:
Bir kaynak programın derlenmesi, şu aşamalardan oluşur:

1. Tarayıcı (kelime analizi yapar, kelime tablosunu üretir)
2. Parser (sözdizimi analizi & anlam analizi yapar, sembol tablosunu üretir ve kontrol amaçlı kullanır)
3. Kod üretici (kod üretimi yapar, makine kodu veya arakod üretir)
4. Optimizasyon (kod optimizasyonu yapar, hız ve bellek kullanımı açısından optimize edilmiş makine kodu üretir)

- Kelime analizinin amacı, dili oluşturan kelimeleri (token) tanımak ve bunları sözdizimi analizi için parser'a iletmektir. 
- Parser, tarayıcıdan ardarda aldığı kelimelere, dilin özel (ayrılmış) kelime ve işaretleri yardımıyla sözdizimsel anlamlar yükler. Bu, bir cümledeki kelimelere özne, nesne, zarf, yüklem gibi görevleri yüklemeye benzer. 
- Bu kelimeler tanındıkça sembol tablosu denen veri yapısına birtakım yan bilgilerle birlikte (değişken tipleri, adresleri, büyüklükleri, vb.) birlikte kaydedilir. Aynı kelimeler tekrar kullanıldıklarında, kullanıldıkları yere göre sembol tablosunda kayıtlı bulunan bilgiler alınarak hata kontrolü yapılır. 
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
- Uç sembollerin kümesi T ile gösterilir. Örneğin, "Mehmet kitabı aldı" cümlesi tarayıcı tarafından parser'a isim(Mehmet), kitabı,aldı şeklinde iletilir. Parantez içindeki Mehmet, isim uç sembolünün taşıdığı özelliktir. 
- Uç sembollerle birlikte parser'a iletilen bu özellikler, parser tarafından üretimler uygulanırken işlenir. 
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
Üretimler u -> v şeklindedir. Burada u,v∈V ve u<>EPSİLON dur.

2. Çevre-bağımlı (context-sensitive) gramerler:
Üretimler uxv -> uvw şeklindedir. Burada u,v,w⊂V, v≠EPSİLON ve x∈N dir. Yani, x ara sembolü, yalnızca u ve w sembol grupları ile çevrelendiği takdirde v sembol grubu ile türetilebilir.

3. Çevre-bağımsız (context-free) gramerler:
Üretimler x->v şeklindedir. Burada v∈V ve xeN dir. Programlama dilleri genellikle bu tür gramerlerden üretilir. Bu tür diller her zaman tanınabilir.

4. Sonlu (veya düzgün) gramerler:
Üretimler x->a veya x->ay şeklindedir. Burada x,y∈N ve a∈T dir.Bu tür gramerler,dilin kelimelerini tanımlamak için kullanılır, yani kelime analizi sırasında tanınan bir gramerdir. Sonlu gramerler, sonlu durum makineleriyle çok etkin şekilde tanınır.

Kelime ve sözdizimi analizinin ayrı ayrı yapılmasının nedeni de, tanınan dillerin farklı gramer sınıflarından olmasıdır. Kelime analizi çok daha etkin algoritmalarla yapılır.

## 4. Gramerlerin Denkliği, Belirsizliği:
L(G) ve L(H) kümeleri birbirine eşitse, G ve H gramerlerine denk gramerler denir. Üretimler farklı sıralarda uygulanıp aynı bir cümle farklı şekilde türetilebiliyorsa bu gramere belirsiz (ambiguous) gramer denir. Parser'ın bu tür belirsiz bir grameri tek şekilde tanıması sağlanmalıdır, aksi halde alınacak sonuçlar her seferinde değişik olacaktır. Örneğin şu gramerde:

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

## 5. Özellikler (attributes) ve Genişletilmiş Gramer:
Her sembolün taşıdığı kendisine ait özellikleri bulunur. Üretimlerin yanında bu özelliklerle ilgili işlemler konarak elde edilen gramere genişletilmiş gramer denir. Sol taraftaki sembolün özelliği, sağda yeralan sembollerin özellikleri işlenerek oluşuyorsa, yani bunların bir fonksiyonuysa:

x -> y1 y2 y3 ... yn ve x.a = f(y1.a, y2.a, y3.a,...,yn.a) ise

(burada x.a ile x'in taşıdığı a özelliği gösteriliyor), bu özelliklere sentezlenmiş özellikler (synthesized attributes) denir.

## 6. Lex ve YACC kullanımı:
UNIX sisteminde mevcut olan Lex ve YACC programları, derleyici oluşturmada çok büyük kolaylıklar sağlar. Lex programı, verilen bir sonlu grameri kabul eden, yani tanıyan tarayıcı üretir. YACC de benzer şekilde, verilen genişletilmiş bir LALR grameri kabul eden ve üretimler sırasında, bunların yanında verilmiş olan sıfatlarla ilgili işlemleri yapan bir parser üretir. İki programın çıktısı da, yani oluşturdukları tarayıcı ve parser programlar C dili ile yazılmıştır.

## 7. Lex'in kabul ettiği (.l uzantılı) dosya yapısı:
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
tamsayı          {sscanf(yytext, "%d", &yylval.ival);return TAMS;}
tamsayı\.tamsayı {sscanf(yytext, "%f", &yylval.fval);return REELS;}
```
Örnekte görüldüğü gibi işlemler `{ }` içinde yeralan C ifadeleridir. `yytext` değişkeni karakter dizisi olup, tanınan kelimenin (örnekte 15, 6349 gibi bir tamsayı veya 0.359, 1368.4 gibi bir reel sayı) saklandığı alandır. Lex tarafından oluşturulan ve parser tarafından çağırılan `yylex()` tarayıcı fonksiyonu tanıdığı tokeni dönerek parser'a iletir. Bunun dışında tokenin özelliğini de `yylval` global değişkeni ile iletir. `{ }` içindeki ifadeler, C bloku içinde yeraldığından, burada lokal C değişkeni tanımlamak da mümkündür. `yylval` değişkeninin tipi, YACC'nin oluşturduğu parser içinde tanımlanmıştır.

## 10. YACC'nin kabul ettiği (.y uzantılı) dosya yapısı:
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

`%left`, `%right`: işlemin sola veya sağa birleşim özelliği varsa `%left işlem` veya `%right işlem` şeklinde belirtilir. Bunlar, gramerdeki belirsizliği gidermek için kullanılır.

`%nonassoc`: işlemin sağa veya sola birleşme özelliği olmadığını belirtir.

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
ifade    : ifade '+' ifade      {$$=$1+$3;}
         | ifade '-' ifade       {$$=$1-$3;}
         | ifade '*' ifade       {$$=$1*$3;}
         | ifade '/' ifade       {$$=$1/$3;}
         | ifade '^' ifade       {$$=power($1,$3);}
         | '-' ifade %prec EKSİ {$$=-$2;}
         | TAMSAYI               {$$=$1;}
         ;
%%
...
```
gramerinde `1+2*3*4^5` cümlesi aşağıdan yukarı doğru şu sırayla türetilir:



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
        <td align="right">/</td>
        <td align="center"></td>
        <td align="left">\</td>
    </tr>
    <tr>
        <td align="center"><strong>1</strong></td>
        <td align="center"></td>
        <td align="center"><strong>2</strong></td>
        <td align="center"></td>
        <td align="center"><strong>3</strong></td>
        <td align="center"></td>
        <td align="center"><strong>4</strong></td>
        <td align="center"></td>
        <td align="center"><strong>5</strong></td>
    </tr>
</tbody></table>






















- Türetme, aynen bu ağaç yapısının postorder taranma sırasını izler. Yani her dal için önce o dalın soldan başlayarak tüm alt dalları taranır, sonra da dalın kendisi taranır. 
- Üretim kurallarının yanında yeralan işlemler, { } içerisinde C ifadeleridir (C bloku). Burada $n şeklindeki hayali değişkenler, sembollerin sıfatlarına karşılık gelirler. Soldaki ara sembolün sıfatı $0 veya $$ ile, sağdaki k.ıncı sembolün sıfatı da $k ile gösterilir. { } içerisinde, diğer işlemlerin yanısıra $1,$2,...,$n işlenerek elde edilen sonuç $$'a atanır. 
- Bu değerler, aslında parser'ın kullandığı stack üzerinde sembollerle birlikte tutularak yukarı doğru işlenerek yolalır, yoksa gerçekte ağaç veri yapısı kullanılmaz. 
- En üst kuralın solundaki sembol, ya da %start ile tanımlanmış başka bir sembol, cümlesel semboldür. Tüm semboller, uç sembollerden başlamak üzere, sağ tarafında yeraldıkları kuralın solundaki ara sembole, aşağıdan yukarı doğru türetilirler. 
- Bu aşamalarda taşınan özellikler kullanılarak ve işlenerek yukarı aşamalara geçirilerek, sonunda cümlesel sembole ulaşıldığında derlenme süreci biter. 
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
statik link	<- B
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
