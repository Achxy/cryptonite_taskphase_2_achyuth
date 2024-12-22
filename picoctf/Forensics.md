# trivial flag transfer protocol
We get a file named `tftp.pcapng`, which is a network capture file, we open it with Wireshark, and are greeted with this:
![Wireshark screen](../assets/contents/trivial_flag_transfer_protocol/1.png)
Few things immediately standout, some files are being transferred via `TFTP` protocol by simply glancing at the `1`st, `19`th and `22`nd record.
![TFTP Packets](../assets/contents/trivial_flag_transfer_protocol/2.png)
Now that it's obvious, let's attempt to extract those files.
We navigate to `File > Export Objects > TFTP` and are met with this screen,
![TFTP object extraction](../assets/contents/trivial_flag_transfer_protocol/3.png)
we save those files to disk to further to analyze them.

The files `instructions.txt` and `plan` seems to be all garbled up.
```sh
achu@air files % cat instructions.txt 
GSGCQBRFAGRAPELCGBHEGENSSVPFBJRZHFGQVFTHVFRBHESYNTGENAFSRE.SVTHERBHGNJNLGBUVQRGURSYNTNAQVJVYYPURPXONPXSBEGURCYNA
achu@air files % cat plan 
VHFRQGURCEBTENZNAQUVQVGJVGU-QHRQVYVTRAPR.PURPXBHGGURCUBGBF 
```
I consult with `dcode.fr` on what the encryption made here could be and it suggested `ROT13`, so we decode it using the same.
```
instructions.txt:
TFTPDOESNTENCRYPTOURTRAFFICSOWEMUSTDISGUISEOURFLAGTRANSFER.FIGUREOUTAWAYTOHIDETHEFLAGANDIWILLCHECKBACKFORTHEPLAN

plan:
IUSEDTHEPROGRAMANDHIDITWITH-DUEDILIGENCE.CHECKOUTTHEPHOTOS
```
Speaking of the program, 
```sh
achu@air files % tar xvf program.deb 
x debian-binary
x control.tar.gz
x data.tar.xz
achu@air files % tar xvf control.tar.gz 
x ./
x ./md5sums
x ./control
achu@air files % cat control
Package: steghide
Source: steghide (0.5.1-9.1)
Version: 0.5.1-9.1+b1
Architecture: amd64
Maintainer: Ola Lundqvist <opal@debian.org>
Installed-Size: 426
Depends: libc6 (>= 2.2.5), libgcc1 (>= 1:4.1.1), libjpeg62-turbo (>= 1:1.3.1), libmcrypt4, libmhash2, libstdc++6 (>= 4.9), zlib1g (>= 1:1.1.4)
Section: misc
Priority: optional
Description: A steganography hiding tool
 Steghide is steganography program which hides bits of a data file
 in some of the least significant bits of another file in such a way
 that the existence of the data file is not visible and cannot be proven.
 .
 Steghide is designed to be portable and configurable and features hiding
 data in bmp, wav and au files, blowfish encryption, MD5 hashing of
 passphrases to blowfish keys, and pseudo-random distribution of hidden bits
 in the container data.
achu@air files % tar xvf data.tar.xz   
x ./
x ./usr/
x ./usr/share/
x ./usr/share/doc/
x ./usr/share/doc/steghide/
x ./usr/share/doc/steghide/ABOUT-NLS.gz
x ./usr/share/doc/steghide/LEAME.gz
x ./usr/share/doc/steghide/README.gz
x ./usr/share/doc/steghide/changelog.Debian.gz
x ./usr/share/doc/steghide/changelog.Debian.amd64.gz
x ./usr/share/doc/steghide/changelog.gz
x ./usr/share/doc/steghide/copyright
x ./usr/share/doc/steghide/TODO
x ./usr/share/doc/steghide/HISTORY
x ./usr/share/doc/steghide/CREDITS
x ./usr/share/doc/steghide/BUGS
x ./usr/share/man/
x ./usr/share/man/man1/
x ./usr/share/man/man1/steghide.1.gz
x ./usr/share/locale/
x ./usr/share/locale/ro/
x ./usr/share/locale/ro/LC_MESSAGES/
x ./usr/share/locale/ro/LC_MESSAGES/steghide.mo
x ./usr/share/locale/fr/
x ./usr/share/locale/fr/LC_MESSAGES/
x ./usr/share/locale/fr/LC_MESSAGES/steghide.mo
x ./usr/share/locale/de/
x ./usr/share/locale/de/LC_MESSAGES/
x ./usr/share/locale/de/LC_MESSAGES/steghide.mo
x ./usr/share/locale/es/
x ./usr/share/locale/es/LC_MESSAGES/
x ./usr/share/locale/es/LC_MESSAGES/steghide.mo
x ./usr/bin/
x ./usr/bin/steghide
achu@air files % 
```
It's highly obvious that `steghide` was used to hide some data in the three images we extracted from `TFTP` traffic.

Using `Aperi'Solve`, we were also get some passwords `3_PHyg2bN!M<1YJ2]Nd`, `DUEDILIGENCE` from previous uploads (connecting the dots the mention of `DUEDILIGENCE` was also made in the `plan` file, so that was a hint). However, the first file did  not have any extractable data.
![Aperisolve](../assets/contents/trivial_flag_transfer_protocol/4.png)
But upon a closer inspection there are several indications of a PGP key(s) of both public and private type and a UNICOS executable in the first image.
```bash
(base) achu@air files % zsteg -a picture1.bmp
imagedata           .. text: "1-!5/'=4&;2'61\"+*#)*\"))(//&.-(//',+%*)!&%$)("
b4,msb,bY           .. text: " Lfd\n*uW"
b8,lsb,bY           .. text: "1-!5/'=4&;2'61\"+*#)*\"))(//&.-(//',+%*)!&%$)("
b8,msb,bY           .. text: "T44T44T444tt4tt"
b7,msb,bY,prime     .. text: "fhd~KD%5"
b8,lsb,bY,prime     .. text: "-54;1+*/.,)&(\"+(.-,!&0,..)).9:5.))/*'&4.2?,I:DGCGACIZ]F:U9a;gG*SrF)m1JCLA\\SM"
b1,rgb,lsb,xY       .. file: OpenPGP Public Key
b2,g,lsb,xY         .. file: OpenPGP Secret Key
b2,rgb,msb,xY       .. file: OpenPGP Secret Key
b3,r,msb,xY         .. text: "+}s8N]Z*"
b3,rgb,lsb,xY       .. text: "djSQbiiP'"
b3p,b,msb,xY        .. file: OpenPGP Public Key
b3p,bgr,msb,xY      .. text: "egn4'\n\n\nYY"
b4,g,lsb,xY         .. text: "7H6p?()l3"
b4,rgb,msb,xY       .. text: ".,`Fj$\nw"
b5,g,lsb,xY         .. file: OpenPGP Secret Key
b5p,g,lsb,xY        .. text: "#Ata\rYdfqN"
b5p,rgb,lsb,xY      .. text: "dddvvIIv"
b5p,bgr,lsb,xY      .. text: "KyTTTff99f"
b6,r,lsb,xY         .. text: "'7dL|vSI$"
b6p,r,lsb,xY        .. text: "agjy.)\"\\XM;^Q"
b6p,g,lsb,xY        .. text: "HC8ssgQto"
b6p,b,lsb,xY        .. text: "WU@6SJ{>4L"
b6p,rgb,lsb,xY      .. text: "44D\ts#OLb5"
b6p,bgr,lsb,xY      .. text: "+\rI:Y/,Q"
b7p,r,lsb,xY        .. text: "[^eTS[WJBGR_ZDXXX]R_Orc]YR_UFJ]\\~"
b7p,g,lsb,xY        .. text: "ekvWS]XMDIQ\\YCWXX]R_Orc]YR_UHM]\\~"
b7p,b,lsb,xY        .. text: "<CMEDLOB;?ITQ?STTYN_Orc]YR_R@DTXz"
b7p,rgb,lsb,xY      .. text: "Z[_iebUUS_Z_VRJQBDFWRU^ZZUEMYaXXX\\\\SS\\__Ocstck\\_XSS`_OTOFLJi\\[\\d"
b7p,bgr,lsb,xY      .. text: "<=COMNEGEQLQNJBI:>>OHKTPPK?GS_TTTXXOOX__Ocstck\\_XSS`_ORI@DDaTUX`{T"
b8,r,lsb,xY         .. text: "--/421**)/-/+)%(!\"#+)*/--*\"&,0,,,..)).//'19:15./,))0/'*'#&%4.-.2?,IH<;893'DGBGHAD@3IcZg]TF=uX9fC8;;]XRmG0SGeZZoNF34)n1VJM^DMMQB\\HSSXM"
b8,g,lsb,xY         .. text: "215=;6+))/./,*&)\"$$+().,,)!%+1,,,..)).//'19:15./,))0/'*'$&&5.-.2?,IJ>=:<3'DHCFG@C?2HbYf\\SC:rU6a>246ZYRgB*MDf^_rQF23(m0UIL]CLLPA[GSTZM"
b8,b,lsb,xY         .. text: "!'&'\"#\"(&('%!$"
b8,rgb,lsb,xY       .. text: "/5!4='2;&16'*+\"*)#))\"//(-.&//(+,')*%%&!()$!\""
b8,rgb,msb,xY       .. text: "44T44T44Ttt4tt4"
b1,g,msb,xY,prime   .. file: OpenPGP Public Key
b2,r,lsb,xY,prime   .. file: OpenPGP Public Key
b2,g,msb,xY,prime   .. file: OpenPGP Public Key
b3p,bgr,lsb,xY,prime.. text: "P\tBUY;^!"
b5p,rgb,msb,xY,prime.. text: "Byzn}8:5"
b5p,bgr,msb,xY,prime.. text: "Svti~0!4"
b6,b,msb,xY,prime   .. text: "@#Z$@,.j\nc"
b6p,r,lsb,xY,prime  .. text: "4=:UFrUQ"
b6p,b,lsb,xY,prime  .. text: "763BAgKG"
b6p,rgb,lsb,xY,prime.. text: "7s=D:*UtG(sHWTR"
b6p,rgb,msb,xY,prime.. text: "og6\tA5m\nI}[f"
b6p,bgr,lsb,xY,prime.. text: "AhC$g<KLF|"
b6p,bgr,msb,xY,prime.. text: "B6F/oYEl"
b7,b,lsb,xY,prime   .. text: "L-nHzX0%"
b7,rgb,lsb,xY,prime .. text: "^E#3oi%s\tA"
b7p,r,lsb,xY,prime  .. text: "^b_EZXb_i"
b7p,g,lsb,xY,prime  .. text: "km^IYXb_k"
b7p,g,msb,xY,prime  .. text: "T$\\tbl,n"
b7p,b,lsb,xY,prime  .. text: "COQ?QTb_`"
b7p,rgb,lsb,xY,prime.. text: "_ibU_RDWZaX\\ct_Oi["
b7p,rgb,msb,xY,prime.. text: "~\nfj;w`iV"
b7p,bgr,lsb,xY,prime.. text: "CONGQJ>OP_TXct_OaU"
b8,r,lsb,xY,prime   .. text: "/41*/)\"+-0,.1:/'4-H9'AIF;RGeZ3\\X"
b8,g,lsb,xY,prime   .. text: "5=6)/*$+,1,.1:/'5-J<'@HC4RBf_2[Z"
b8,g,msb,xY,prime   .. text: "L|l~a):<"
b8,b,lsb,xY,prime   .. text: "'(/*,1:/'0*F7%<D=5LCa[.WT"
b8,rgb,lsb,xY,prime .. text: "/5!4='16'*)#//()*%\"$"
b8,bgr,lsb,xY,prime .. text: "!5/'=4'61#)*(//%*)"
b2,b,lsb,xy         .. text: "4QLM\t|#,C6m"
b2,rgb,msb,xy       .. file: OpenPGP Public Key
b2,bgr,lsb,xy       .. file: OpenPGP Secret Key
b2,bgr,msb,xy       .. text: ":fYVoYC?"
b3,bgr,msb,xy       .. text: "7&ji=D4aB"
b3p,r,msb,xy        .. text: "hGBFgL7G"
b3p,g,lsb,xy        .. text: " F*GoH*%"
b3p,rgb,lsb,xy      .. text: "WwwwRRT03))P--RR"
b4,r,lsb,xy         .. file: OpenPGP Public Key
b4,r,msb,xy         .. file: OpenPGP Public Key
b4,g,lsb,xy         .. text: "xvESD#31"
b4,rgb,lsb,xy       .. file: OpenPGP Secret Key
b5,rgb,lsb,xy       .. text: "_]qC)=?aG("
b5p,r,lsb,xy        .. text: "AO=/,6$$+"
b5p,g,lsb,xy        .. text: "lolvddkZkT]mledRc\\aHIH8>%+$"
b5p,b,lsb,xy        .. text: "$-?IRTSRccdcmlvevmlmcdddc[\\mftmwx"
b5p,rgb,lsb,xy      .. text: "@OH>=,-?-$66$$$$-"
b5p,bgr,lsb,xy      .. text: "`_`P-HVMDa]"
b6,g,msb,xy         .. text: "0,MS,M14"
b6,b,lsb,xy         .. file: OpenPGP Public Key
b6,bgr,lsb,xy       .. file: OpenPGP Public Key
b6,bgr,msb,xy       .. file: OpenPGP Public Key
b6p,r,lsb,xy        .. text: "/3000;FPUc`bjennonpppkponoheeed`aghroqvz{|"
b6p,g,lsb,xy        .. text: "BbG2tEFE!n"
b6p,g,msb,xy        .. text: "\n\nJ}R\nN:"
b6p,b,lsb,xy        .. file: OpenPGP Public Key
b6p,rgb,lsb,xy      .. text: "zpukdhbb"
b6p,bgr,lsb,xy      .. file: OpenPGP Public Key
b6p,bgr,msb,xy      .. file: OpenPGP Public Key
b7,g,msb,xy         .. text: "A'S(d\n\rs"
b7p,r,lsb,xy        .. text: "~~y{xuwvuvxsv|}|"
b7p,g,lsb,xy        .. text: "|yplhc[ZXTROOLKHDNF>A@:C"
b7p,b,lsb,xy        .. text: "}ztrpljffge^\\ibVURRY"
b7p,rgb,lsb,xy      .. text: "|x{{wxxtwwwwuuvvxxxsww}|||"
b7p,bgr,lsb,xy      .. text: "|z{yuusppllhkhgagifbec^`\\\\i"
b8,r,lsb,xy         .. text: "pjR]U_QHJQPHVy]Y\\MON@K@AdRUTX_/Nd5CNPgvyMqzyqn"
b8,g,lsb,xy         .. file: OpenPGP Secret Key
b8,b,lsb,xy         .. text: "fhgwhmco"
b8,rgb,lsb,xy       .. text: "pvijq`RXI]cVU]M_gZQYJHP@JRCQXJPVFHQCVcKy"
b8,bgr,lsb,xy       .. text: "ivp`qjIXRVc]M]UZg_JYQ@PHCRJJXQFVPCQHKcVg"
b3,r,lsb,xy,prime   .. file: PGP Secret Sub-key -
b3,rgb,lsb,xy,prime .. text: "PZku ~!L]S"
b3p,r,lsb,xy,prime  .. file: PGP Secret Sub-key -
b5p,bgr,lsb,xy,prime.. text: "<PYbciXaXr"
b6,b,lsb,xy,prime   .. text: "WYvWY%]'"
b6p,r,lsb,xy,prime  .. text: ")(+}QcjakY"
b6p,b,lsb,xy,prime  .. text: "kW\\]LAAYe\t"
b6p,rgb,lsb,xy,prime.. text: "(2'(2B\r'"
b6p,bgr,lsb,xy,prime.. text: "bk\\W]]a\\WNCCEAF[vg&"
b7,g,msb,xy,prime   .. file: zlib compressed data
b7,b,msb,xy,prime   .. text: "?x\".W$fp"
b7p,r,lsb,xy,prime  .. text: "26628?O]"
b7p,g,lsb,xy,prime  .. text: "0AEBBDLWk"
b7p,b,lsb,xy,prime  .. text: "E:F9&GUX[Y\\`oy%}"
b7p,rgb,lsb,xy,prime.. text: "lXT<:?8,"
b7p,bgr,lsb,xy,prime.. text: "zhhibX?IJ"
b8,r,lsb,xy,prime   .. text: "pRQJVO@U/dPzs"
b8,g,lsb,xy,prime   .. text: "vXYRcQF^:mY"
b8,b,lsb,xy,prime   .. text: "gwmoiIJCKG:J2UIws"
b8,rgb,lsb,xy,prime .. text: "opviRXIQYJJRCVcKOQG@F:U^J/:2dmUPYIz"
b8,bgr,lsb,xy,prime .. text: "ivpIXRJYQCRJKcVGQO:F@J^U2:/UmdIYPw"
b8,bgr,msb,xy,prime .. text: "uE9\r%y\re"
b1,b,msb,yx         .. file: OpenPGP Secret Key
b2,bgr,lsb,yx       .. file: OpenPGP Secret Key
b3,r,lsb,yx         .. text: "t&r1\\so}"
b3p,rgb,msb,yx      .. text: "VVO%%w*U"
b4,rgb,lsb,yx       .. file: OpenPGP Secret Key
b5,g,msb,yx         .. text: "tt\"8hbD/"
b5p,b,lsb,yx        .. text: "\r\rD{]2RJ"
b5p,rgb,lsb,yx      .. text: "#Guc=K\\JSq"
b6,b,lsb,yx         .. file: OpenPGP Public Key
b6,bgr,lsb,yx       .. file: OpenPGP Public Key
b6,bgr,msb,yx       .. file: OpenPGP Public Key
b6p,r,lsb,yx        .. text: "YBmdkbzv"
b6p,b,lsb,yx        .. file: OpenPGP Public Key
b6p,b,msb,yx        .. file: OpenPGP Public Key
b6p,rgb,lsb,yx      .. text: "\\Ps*EJE E`"
b6p,bgr,lsb,yx      .. file: OpenPGP Public Key
b6p,bgr,msb,yx      .. file: OpenPGP Public Key
b7p,r,lsb,yx        .. text: "noVINLGIVDMIR-LZB-]"
b7p,g,lsb,yx        .. text: "}{bSRRSUbP]Yb=\\jQ:`"
b7p,b,lsb,yx        .. text: "udyUYSnIFejZX_IDIA?AL9D8E"
b7p,rgb,lsb,yx      .. text: "W_W\\wi}y"
b7p,bgr,lsb,yx      .. text: "eyxRUVYWROoyIOGLeSk]ZmXU_WIFEII6AV?G@CMD8FD%8EE&"
b8,r,lsb,yx         .. text: "cO:Uun_TM[hOHJKz_W\\U_IpBKG3455764KR+/+.;4><FO7673+&$'' &0#'$%+(\"(&"
b8,g,lsb,yx         .. file: OpenPGP Secret Key
b8,b,lsb,yx         .. text: "fMK:Od^OE?JT?898kG>C@N:a2<<)*+,+)'7<$'#&2)5.-6,*/+$#\"$$"
b8,rgb,lsb,yx       .. text: "fcpMOZK:D:U`Ou}dnv^_gOT\\EMV?[eJhsTOY?HS8JS9KT8z"
b8,bgr,lsb,yx       .. text: "MpcKZO:D:O`Ud}u^vnOg_E\\T?VMJe[Tsh?YO8SH9SJ8TKk"
b1,rgb,msb,yx,prime .. file: OpenPGP Secret Key
b3,rgb,msb,yx,prime .. file: PGP Secret Sub-key -
b3p,r,msb,yx,prime  .. file: PGP Secret Sub-key -
b5p,g,lsb,yx,prime  .. text: "<>$(#.t>"
b5p,rgb,lsb,yx,prime.. text: "XZ>ZZFSQ"
b6,r,msb,yx,prime   .. text: "1A7X6;E>"
b6p,r,lsb,yx,prime  .. text: "~y;Wzxqd"
b6p,b,lsb,yx,prime  .. text: "~}lnGhmv"
b6p,bgr,lsb,yx,prime.. text: "Foh\"lFwz"
b6p,bgr,msb,yx,prime.. text: "_chY{7-5I}Zb"
b7,b,lsb,yx,prime   .. file: OpenPGP Secret Key
b7,bgr,lsb,yx,prime .. file: PGP Secret Sub-key -
b7p,r,lsb,yx,prime  .. text: "KGEC,JIQ!6#"
b7p,g,lsb,yx,prime  .. text: "QSQJ=XX^'B3"
b7p,b,lsb,yx,prime  .. file: OpenPGP Secret Key
b7p,rgb,lsb,yx,prime.. text: "jo_iymOOK34"
b7p,rgb,msb,yx,prime.. text: "1vEVA1AN"
b7p,bgr,lsb,yx,prime.. file: OpenPGP Secret Key
b7p,bgr,msb,yx,prime.. text: "z6BRARRB"
b8,r,lsb,yx,prime   .. file: Solitaire Image Recorder format
b8,g,lsb,yx,prime   .. text: "ZD}gsSrix>;7=C=)-+!\"S)(!)+("
b8,b,lsb,yx,prime   .. text: "K:dOT8GCa++').*$#!"
b8,rgb,lsb,yx,prime .. text: "OZK:D:u}d_gOhsTHS8_rG\\iCpxa5>+7;+/7'4=)<C.6=*')$'-#%+!"
b8,bgr,lsb,yx,prime .. text: "KZO:D:d}uOg_Tsh8SHGr_Ci\\axp+>5+;7'7/)=4.C<*=6$)'#-'!+%"
b1,r,lsb,XY         .. text: "Va\"DP{2Ii"
b1,g,msb,XY         .. file: OpenPGP Public Key
b1,rgb,lsb,XY       .. text: "6C\n-dv;2"
b3,bgr,msb,XY       .. text: "uGE6&&$a"
b4,rgb,lsb,XY       .. file: OpenPGP Public Key
b5,rgb,lsb,XY       .. text: "-W{{\r$80*=C"
b5p,rgb,lsb,XY      .. text: "#+4sfh(F"
b5p,rgb,msb,XY      .. text: "Gk/\\|jLa"
b5p,bgr,msb,XY      .. text: "Qv1C}cUh"
b6,r,msb,XY         .. text: "nO8usv/}"
b6p,r,lsb,XY        .. text: "R_9LX^!*-zke`"
b6p,g,lsb,XY        .. text: "luSgsp;@H"
b6p,g,msb,XY        .. file: OpenPGP Secret Key
b6p,b,lsb,XY        .. text: "L6?zKR4AWU"
b6p,bgr,msb,XY      .. text: "i%!is-u^"
b7,r,lsb,XY         .. text: "_=)/i/VPbD"
b7p,r,lsb,XY        .. text: "YJrihHl]["
b7p,g,lsb,XY        .. text: "4z-X+)^>"
b7p,b,lsb,XY        .. text: "`fKCgVL,WKI"
b7p,rgb,lsb,XY      .. text: "XZJPrhhNhaH]m]\\W[f"
b7p,bgr,lsb,XY      .. text: "JRBJfZV<LA,EWKJCIR"
b7p,bgr,msb,XY      .. text: "RJBRfZj<2"
b8,r,lsb,XY         .. text: "Y\\`[EE_g[YTq"
b8,r,msb,XY         .. file: OpenPGP Public Key
b8,g,lsb,XY         .. text: "hosnXZsylie"
b8,g,msb,XY         .. text: "Rb$LllL|"
b8,b,lsb,XY         .. text: ">CA>+,EKA@8L\\FLOHT`KEONHHKOPA@Z84H6I"
b8,rgb,lsb,XY       .. text: "Yh>\\oC`sA[n>EX+EZ,_sEgyK[lAYi@Te8q"
b8,rgb,msb,XY       .. file: OpenPGP Public Key
b8,bgr,lsb,XY       .. text: ">hYCo\\As`>n[+XE,ZEEs_KygAl[@iY8eTL"
b1,r,lsb,XY,prime   .. text: " 8p\"s+fX"
b1,g,lsb,XY,prime   .. file: PGP Secret Sub-key -
b1,b,lsb,XY,prime   .. text: "i^\\\na&}O"
b2,g,msb,XY,prime   .. file: PGP Secret Sub-key -
b2,b,msb,XY,prime   .. file: OpenPGP Public Key
b5,g,lsb,XY,prime   .. file: OpenPGP Public Key
b5,b,lsb,XY,prime   .. text: "@1/I>(]3"
b6,g,lsb,XY,prime   .. text: "_vF-1dX$"
b6,g,msb,XY,prime   .. text: "f4iBfhlr"
b6p,g,lsb,XY,prime  .. text: "wDuxp,n2"
b6p,rgb,lsb,XY,prime.. text: "`HnRAT9=S8j"
b6p,bgr,lsb,XY,prime.. text: "L4ZF=L99O4R"
b7p,r,lsb,XY,prime  .. text: "U2PCVG90"
b7p,g,lsb,XY,prime  .. text: "lAaPfQG?%"
b7p,b,lsb,XY,prime  .. text: "xvEcY>Ot~vKT3`a>,BtZ"
b7p,rgb,lsb,XY,prime.. text: "gLZg{tT4~J"
b7p,bgr,lsb,XY,prime.. text: "pe}eR<KC"
b8,r,lsb,XY,prime   .. text: "`[EgqgzokXP8SE-'.+c>5HE3&-3=:*"
b8,g,lsb,XY,prime   .. text: "}xf^CaS82;7kG<UU?*:=IF6"
b8,b,lsb,XY,prime   .. text: "A>,KLFTKH@82>2)"
b8,rgb,lsb,XY,prime .. text: "`sA[n>EZ,gyKq"
b8,bgr,lsb,XY,prime .. text: "As`>n[,ZEKygL"
b2,bgr,lsb,YX       .. text: "qo=&14\n;"
b2,bgr,msb,YX       .. text: ")OfB@Qaql"
b3p,b,msb,YX        .. file: OpenPGP Public Key
b4,rgb,lsb,YX       .. file: OpenPGP Public Key
b5,rgb,msb,YX       .. text: "|Ga%(h90t"
b5p,b,lsb,YX        .. text: "Lwgb\\9\"np"
b5p,bgr,lsb,YX      .. text: "zKdt=g>fR_"
b6,rgb,lsb,YX       .. text: "|]'<)o\tS"
b6p,r,lsb,YX        .. text: "}egznZ@7'"
b6p,g,lsb,YX        .. text: "dVY05V}2"
b6p,rgb,lsb,YX      .. text: ">(\\DeU#O."
b7p,r,lsb,YX        .. text: ":oNL]1E5A&\"5"
b7p,g,lsb,YX        .. text: "%onCPeZU<X{]i}NcR\\BEW"
b7p,rgb,lsb,YX      .. text: "W9Wz/966H1;06Q"
b8,r,lsb,YX         .. text: "NXpnmWXOkKXXaAvs"
b8,r,msb,YX         .. file: OpenPGP Public Key
b8,g,lsb,YX         .. text: "|x`bYxXcdmI"
b8,b,lsb,YX         .. text: ">F9b7=RRQFC:V6A9E*UThDNW>/C>JFNJ:E9VY>W<<:6@<DLK`B+<6=O=SQE=K[Z\\FJ[T@2NTYTZX`R]aXYiV\\ieq\\l\\ALKkT_j_VWZh\\WWz|;][YcliSe]ZuYVVLhq^wth"
b8,rgb,lsb,YX       .. text: "Yh>_nFUg9"
b8,rgb,msb,YX       .. file: OpenPGP Public Key
b8,bgr,lsb,YX       .. text: ">hYFn_9gUb"
b8,bgr,msb,YX       .. text: "<:J<jR\\*"
b1,r,msb,YX,prime   .. text: "Vp 7\n|f9"
b3p,r,msb,YX,prime  .. text: "Y,R^hg))"
b3p,b,msb,YX,prime  .. file: OpenPGP Secret Key
b4,bgr,lsb,YX,prime .. file: PGP Secret Sub-key -
b5,r,lsb,YX,prime   .. text: "P_ef]ItW&o0c+"
b5,g,lsb,YX,prime   .. text: "Sg7\"/\t[t"
b5,rgb,msb,YX,prime .. file: OpenPGP Secret Key
b5p,rgb,msb,YX,prime.. text: "OfXxt&R3"
b5p,bgr,lsb,YX,prime.. text: "N`P)k4?E6"
b5p,bgr,msb,YX,prime.. text: "VkFvb*C;"
b6,bgr,lsb,YX,prime .. text: "ubL/fa/."
b7p,g,lsb,YX,prime  .. text: "(;RokaR]2JI-A`Ob"
b7p,rgb,lsb,YX,prime.. text: "/996Q'Q?<%+"
b8,r,lsb,YX,prime   .. text: "XnOKAsyQUXIQ^JCd^jhne"
b8,g,lsb,YX,prime   .. text: "`eaTYeRTvkwu}s"
b8,g,msb,YX,prime   .. text: "QnUac3R="
b8,b,lsb,YX,prime   .. text: "9b=R:6*TWFJ>:@K==[TTTaVljZ\\|]lwQ|"
b8,rgb,lsb,YX,prime .. text: "bXf=n|ROY:KX6AI*s"
b8,rgb,msb,YX,prime .. text: "j!\t6^QVF"
b8,bgr,lsb,YX,prime .. text: "=fXR|n:YO6XK*IAT"
b1,b,lsb,Xy         .. text: "g*Y\\HUb]E"
b2,b,lsb,Xy         .. file: OpenPGP Secret Key
b2,rgb,lsb,Xy       .. text: "+SS|QdS8"
b2,bgr,lsb,Xy       .. text: "l|sI_T\\?"
b3p,r,msb,Xy        .. text: "x8~a|pPxE"
b3p,g,lsb,Xy        .. text: "%%\t!&DKo*F("
b3p,g,msb,Xy        .. text: "\nQz\t\t\"NK1"
b4,g,lsb,Xy         .. text: "2r$CTgux"
b5,b,msb,Xy         .. text: ":Qb6u`#/"
b5,rgb,msb,Xy       .. text: ",3q}*C1m"
b5p,r,msb,Xy        .. text: " @=0\tYY{"
b5p,g,lsb,Xy        .. text: "/\tcSI@O?"
b5p,b,lsb,Xy        .. text: "ulvel[[dddcelmnuftmkdccbSTRQO=,#"
b5p,rgb,lsb,Xy      .. text: "iD;2;))  )"
b5p,bgr,lsb,Xy      .. text: "{{rii`rr`hjaXXXXaaaaaaaXbii`iiirri`rrckkkYbbbYbY"
b6,bgr,msb,Xy       .. file: OpenPGP Secret Key
b6p,r,lsb,Xy        .. text: "zzusnpke``eeedknolshppronnmfj`caTRG80033/"
b6p,g,lsb,Xy        .. text: "zy|medbZZYYL"
b6p,b,lsb,Xy        .. text: "(0Acy__^MFEDE>GJE<@=DJGQMPTUZUag`Z___U]Z"
b6p,b,msb,Xy        .. file: OpenPGP Secret Key
b6p,rgb,lsb,Xy      .. text: "?5?::JlC"
b6p,bgr,lsb,Xy      .. text: "2+?5?>:Jp?"
b7p,r,lsb,Xy        .. text: "QMKHKG7NI@?<<;96853003332567694888766724011*(#"
b7p,g,lsb,Xy        .. text: "YUSPSOB^VOLKJGEBD@E@@CCCBEBCBE@EGGDEEDAB<=>7320,,--&"
b7p,b,lsb,Xy        .. text: "]\"ommii`Rllc`_^_]Z\\YZWWXXXYYZ[Z]X\\[[XYYXUTTUSOJID@@AA9"
b7p,rgb,lsb,Xy      .. text: ">YYeib[UTG"
b7p,bgr,lsb,Xy      .. text: "Pkkw{tmed_"
b8,r,lsb,Xy         .. text: "PB&:ilTB -.(2KC"
b8,r,msb,Xy         .. text: "Y999999Y"
b8,g,lsb,Xy         .. text: "XJ,?pvZG%23-7PH$117963/0-kVE588CGT`d"
b8,b,lsb,Xy         .. text: "ZM1Etz[N(671;TL(55;=:622/nYI9<<GKXdh"
b8,rgb,lsb,Xy       .. text: "PXZBJM&,1:?EiptlvzTZ[BGN %(-26.37(-127;KPTCHL"
b8,bgr,lsb,Xy       .. text: "ZXPMJB1,&E?:tpizvl[ZTNGB(% 62-73.1-(;72TPKLHC($"
b8,bgr,msb,Xy       .. text: "sssssKKK;;"
b1,g,msb,Xy,prime   .. text: "Rr,.ixG:"
b1,b,msb,Xy,prime   .. text: "R!s:]Bsx"
b1,bgr,lsb,Xy,prime .. file: OpenPGP Public Key
b3p,b,msb,Xy,prime  .. text: "HR2+[]\rq1"
b3p,rgb,lsb,Xy,prime.. text: "@DI$#hhGG"
b5p,g,lsb,Xy,prime  .. text: "U2Jccoy?"
b5p,rgb,lsb,Xy,prime.. text: "pf^Ui/KT{T"
b5p,bgr,lsb,Xy,prime.. text: "iXaairibPV)!"
b6,r,lsb,Xy,prime   .. file: OpenPGP Public Key
b6,rgb,lsb,Xy,prime .. file: OpenPGP Public Key
b6,bgr,lsb,Xy,prime .. file: OpenPGP Public Key
b6p,r,lsb,Xy,prime  .. file: OpenPGP Public Key
b6p,g,lsb,Xy,prime  .. text: "__?@WRX&{"
b6p,b,lsb,Xy,prime  .. file: OpenPGP Secret Key
b6p,rgb,lsb,Xy,prime.. file: OpenPGP Public Key
b6p,bgr,lsb,Xy,prime.. text: "&0o]FENJCTUWX]Vk\\"
b7,b,msb,Xy,prime   .. file: OpenPGP Public Key
b7p,r,lsb,Xy,prime  .. text: "bH@4374("
b7p,g,lsb,Xy,prime  .. text: "BCBB@3-*y"
b7p,g,msb,Xy,prime  .. file: OpenPGP Public Key
b7p,b,lsb,Xy,prime  .. text: "[XZZTJA?"
b7p,b,msb,Xy,prime  .. file: OpenPGP Public Key
b7p,rgb,lsb,Xy,prime.. text: "YiTf}g[^a"
b7p,bgr,lsb,Xy,prime.. text: "`[VYY[\\[XTTKI@??"
b7p,bgr,msb,Xy,prime.. text: "9/]\rscp\t"
b8,r,lsb,Xy,prime   .. text: "&:lB(K,4*3>"
b8,g,lsb,Xy,prime   .. text: ",?vG-P1908C"
b8,b,lsb,Xy,prime   .. text: "1EzN1T5=2<G"
b8,rgb,lsb,Xy,prime .. text: "&,1:?ElvzBGN(-1KPT,1549=*0238<>CG"
b8,bgr,lsb,Xy,prime .. text: "1,&E?:zvlNGB1-(TPK51,=9420*<83GC>"
b8,bgr,msb,Xy,prime .. text: "*rMuUMMM"
b1,g,msb,yX         .. file: OpenPGP Public Key
b1,b,msb,yX         .. text: "l>h/Xm*y"
b1,bgr,lsb,yX       .. text: "kovo:19~"
b1,bgr,msb,yX       .. file: OpenPGP Public Key
b3,r,lsb,yX         .. file: SVr4 curses screen image, big-endian
b3,g,msb,yX         .. file: zlib compressed data
b3p,r,msb,yX        .. text: "rPW\rA\t97"
b5,b,msb,yX         .. text: "\rwFz`tpjy-"
b5p,g,msb,yX        .. file: Amiga Workbench icon, vers. 1277
b5p,b,lsb,yX        .. text: "5--)S[dulT"
b5p,rgb,lsb,yX      .. text: "45GOPQZc[uu"
b5p,bgr,lsb,yX      .. text: "%0(--.-,KV[]]ffpohfV"
b6,g,lsb,yX         .. text: "}at\rkC@|PCTLp"
b6,g,msb,yX         .. file: unicos (cray) executable
b6,bgr,msb,yX       .. file: OpenPGP Secret Key
b6p,r,lsb,yX        .. text: "-@Jkzmeg"
b6p,g,msb,yX        .. file: OpenPGP Public Key
b6p,b,lsb,yX        .. text: "1<3B*?#7"
b6p,b,msb,yX        .. file: OpenPGP Secret Key
b6p,rgb,lsb,yX      .. text: "%/5CCKYhmxxmefed"
b6p,bgr,lsb,yX      .. text: "i9E@D%f6q"
b7p,r,lsb,yX        .. text: "K8LRUY\\diy"
b7p,g,lsb,yX        .. text: "`H[^bgksp"
b7p,b,lsb,yX        .. text: "hU2o]lqwx|"
b7p,rgb,lsb,yX      .. text: "8LMQSTUVYW]]eehny"
b7p,bgr,lsb,yX      .. text: "Gi!Uj2=n"
b8,r,lsb,yX         .. text: "P8:1>%QS[J>0y"
b8,g,lsb,yX         .. text: "X?A8D(WW`NC4}"
b8,b,lsb,yX         .. text: "ZAD:H.\\\\cRF7"
b8,rgb,lsb,yX       .. text: "PXZ8?A:AD18:>DH%(.QW\\SW\\[`cJNR>CF047y}"
b8,bgr,lsb,yX       .. text: "ZXPA?8DA::81HD>.(%\\WQ\\WSc`[RNJFC>740"
b1,g,lsb,yX,prime   .. file: OpenPGP Secret Key
b1,rgb,lsb,yX,prime .. text: "58%]_(-R"
b2,r,lsb,yX,prime   .. file: PGP Secret Sub-key -
b2,b,lsb,yX,prime   .. text: "K2y<O;8k)"
b3,r,msb,yX,prime   .. text: "Db/2:|Np|"
b5,g,lsb,yX,prime   .. text: "%t/f\n8>8"
b5,rgb,msb,yX,prime .. text: "_`=|}IVO"
b5p,b,lsb,yX,prime  .. text: "1SBJa%W9M`y"
b5p,rgb,msb,yX,prime.. text: "Mi.$b}Xd"
b5p,bgr,msb,yX,prime.. text: "^z27tuPe"
b6,g,msb,yX,prime   .. text: "td\tb 7\"8"
b6p,r,msb,yX,prime  .. file: PGP Secret Sub-key -
b6p,rgb,lsb,yX,prime.. text: "5CmxeI>B"
b6p,rgb,msb,yX,prime.. file: PGP Secret Sub-key -
b6p,bgr,msb,yX,prime.. text: "}u%/hk`K"
b7,b,lsb,yX,prime   .. text: "+U(~8Qa B"
b7,rgb,lsb,yX,prime .. text: "*92d;}\t\"E*"
b7p,g,lsb,yX,prime  .. text: "(wfS4S|XPs\\D"
b7p,rgb,lsb,yX,prime.. text: "\\J-<=%kGF"
b8,r,lsb,yX,prime   .. text: "YaQ:>/M)"
b8,g,lsb,yX,prime   .. text: "[eW@C4K."
b8,g,msb,yX,prime   .. text: "Ow73a~R~"
b8,b,lsb,yX,prime   .. text: "\\aO:=2Y0"
b8,rgb,lsb,yX,prime .. text: ":AD18:%(.SW\\047"
b8,bgr,lsb,yX,prime .. text: "DA::81.(%\\WS740"
b2,bgr,msb,Yx       .. text: "}H7#3bjjf7"
b3p,b,msb,Yx        .. file: OpenPGP Public Key
b3p,rgb,msb,Yx      .. text: "U*w%%OVV"
b5,g,lsb,Yx         .. file: OpenPGP Secret Key
b5p,r,lsb,Yx        .. text: "rWVLQnAjJ_w#_cS"
b5p,g,lsb,Yx        .. file: OpenPGP Secret Key
b5p,b,lsb,Yx        .. text: "5EN(eufz"
b5p,rgb,lsb,Yx      .. text: "Q>Q5HdSJo"
b5p,bgr,lsb,Yx      .. text: "3*G,N6*C`ipke"
b6,rgb,msb,Yx       .. text: "iY%m_nsWd"
b6p,r,lsb,Yx        .. text: "DFFp\\`9kl"
b6p,r,msb,Yx        .. text: "/[K94oo;["
b6p,b,lsb,Yx        .. text: "HhbJ\\Sm22"
b6p,rgb,lsb,Yx      .. text: "T3DN) )vi"
b6p,bgr,lsb,Yx      .. text: "kiduB^]=Xn["
b7p,r,lsb,Yx        .. text: "zz_|\\q}r"
b7p,g,lsb,Yx        .. text: "fnleqq{vXa"
b7p,b,lsb,Yx        .. text: "<;EcMQRJY]X^BETTKWHkKbri|"
b7p,rgb,lsb,Yx      .. text: "]}fhllw{"
b7p,bgr,lsb,Yx      .. text: "<<;BENcGMJQJRLJQX[\\[Y=_DB@DSUTUMJWW_IQjBJCcdshh[|"
b7p,bgr,msb,Yx      .. text: "RJvZB&6RNz"
b8,r,lsb,Yx         .. text: "-//23;L31060QXQS`^bK?.>3466;=@A==B=:/3>*.(87>992CII&),0?B0/,5=N\"*'*&),*)-6(1-*)*+"
b8,g,lsb,Yx         .. text: "24367<I532746623898;=+;.,.0;ADEABGA;15@,0)98?::3DJI&),0?B0/+4<M!)&)%(,+*/3&0,)()*"
b8,b,lsb,Yx         .. text: "!\"'1#&%(%)&%(,-.-,"
b8,rgb,lsb,Yx       .. text: "26!37\";<'LI135#13&02%67(04%Q6)X6&Q2%S3(`8,^9-b8.K;-?=,.+"
b8,bgr,lsb,Yx       .. text: "3/!62\"73'<;1IL#53&31%20(76%40)6Q&6X%2Q(3S,8`-9^.8b-;K,=?"
b1,r,lsb,Yx,prime   .. text: ")nZ\t|wBoy"
b1,g,lsb,Yx,prime   .. text: "zifm43mx"
b2,r,lsb,Yx,prime   .. text: "bLpR1k\\7@"
b3,r,msb,Yx,prime   .. file: PGP Secret Sub-key -
b3,bgr,lsb,Yx,prime .. text: "z0R,$P#?"
b4,g,lsb,Yx,prime   .. text: "K0dCL@JL"
b4,rgb,msb,Yx,prime .. text: "t\"?m* 0W"
b4,bgr,msb,Yx,prime .. text: "/b= : Xw"
b5,b,lsb,Yx,prime   .. text: "`2.NAf9+"
b5p,g,lsb,Yx,prime  .. text: "z4fv&\ry@4"
b5p,b,msb,Yx,prime  .. file: PGP Secret Sub-key -
b5p,bgr,lsb,Yx,prime.. text: "\r\"s!N*!Nl*z~"
b5p,bgr,msb,Yx,prime.. text: "U}53~te-r"
b6p,b,lsb,Yx,prime  .. text: "m`B#n3E:eViZNj"
b6p,bgr,lsb,Yx,prime.. text: "`3AX >lH3]D"
b7,rgb,lsb,Yx,prime .. text: "$J\r&Xy{Ui"
b7p,r,lsb,Yx,prime  .. text: "f{QdYMl_Ul}"
b7p,g,lsb,Yx,prime  .. text: "RfVJg\\Rht"
b7p,b,lsb,Yx,prime  .. text: ";OJ[DLB[NB\\SI[_qPZMe[70 "
b7p,rgb,lsb,Yx,prime.. text: "{gQneXYzMRmU^VU^lX|"
b7p,bgr,lsb,Yx,prime.. text: ";BNGJL[[DTMQCd[FOpCH]IRJIRZJ^^p.Qa[PMse"
b8,r,lsb,Yx,prime   .. text: "/2;30X^K3@=3(72,,=&)6*/+*/6,>AG#39<72B8G9E$2("
b8,g,lsb,Yx,prime   .. text: "36<5469;.DA5)83,+<%*3).*).4*:>H$7;50/A7G6B*;0"
b8,g,msb,Yx,prime   .. text: "Td$4xpd$h"
b8,b,lsb,Yx,prime   .. text: "!'#%&--\"*&(!2-#'8!$.$)%$)-%//8"
b8,rgb,lsb,Yx,prime .. text: "26!;<'35#04%X6&^9-K;-3.\"@D*=A&35(()!78223-,,#,+'=<8&%!)*$63.*)$/.)+*%*)$/.)64-,*%>:/A>/GH8\#$"
b8,bgr,lsb,Yx,prime .. text: "3/!62'<;#53%40&6X-9^-;K\".3*D@&A=(53!)(287-32#,,'+,8<=!%&$*).36$)*)./%*+$)*)./-46%*,/:>/>A8HG"
(base) achu@air files % 
```
!TODO


# tunn3l v1s10n
We get a file named `tunn3l_v1s10n`, now we have to figure out what it is and how to get our flag from it.
```bash
achu@air ~ % file /Users/achu/Downloads/tunn3l_v1s10n 
/Users/achu/Downloads/tunn3l_v1s10n: data
achu@air ~ % binwalk /Users/achu/Downloads/tunn3l_v1s10n 
Analyzed 1 file for 85 file signatures (187 magic patterns) in 27.0 milliseconds
achu@air ~ % sf /Users/achu/Downloads/tunn3l_v1s10n    
---
siegfried   : 1.11.1
scandate    : 2024-11-04T00:54:02+05:30
signature   : default.sig
created     : 2024-06-21T16:01:11+10:00
identifiers : 
  - name    : 'pronom'
    details : 'DROID_SignatureFile_V118.xml; container-signature-20240501.xml'
---
filename : '/Users/achu/Downloads/tunn3l_v1s10n'
filesize : 2893454
modified : 2024-11-04T00:27:44+05:30
errors   : 
matches  :
  - ns      : 'pronom'
    id      : 'UNKNOWN'
    format  : 
    version : 
    mime    : 
    class   : 
    basis   : 
    warning : 'no match'
```
`TRid` did not return any results either.
we're off to a spectacularly bad start...
Let's see the hex dump
```bash
achu@air ~ % xxd /Users/achu/Downloads/tunn3l_v1s10n 
00000000: 424d 8e26 2c00 0000 0000 bad0 0000 bad0  BM.&,...........
00000010: 0000 6e04 0000 3201 0000 0100 1800 0000  ..n...2.........
00000020: 0000 5826 2c00 2516 0000 2516 0000 0000  ..X&,.%...%.....
00000030: 0000 0000 0000 231a 1727 1e1b 2920 1d2a  ......#..'..) .*
00000040: 211e 261d 1a31 2825 352c 2933 2a27 382f  !.&..1(%5,)3*'8/
00000050: 2c2f 2623 332a 262d 2420 3b32 2e32 2925  ,/&#3*&-$ ;2.2)%
00000060: 3027 2333 2a26 382c 2836 2b27 392d 2b2f  0'#3*&8,(6+'9-+/
00000070: 2623 1d12 0e23 1711 2916 0e55 3d31 9776  &#...#..)..U=1.v
00000080: 668b 6652 996d 569e 7058 9e6f 549c 6f54  f.fR.mV.pX.oT.oT
00000090: ab7e 63ba 8c6d bd8a 69c8 9771 c193 71c1  .~c..m..i..q..q.
000000a0: 9774 c194 73c0 9372 c08f 6fbd 8e6e ba8d  .t..s..r..o..n..
```
Search wikipedia for matching magic number/binary signature
![Wikipedia binary signature info](../assets/contents/tunn3l_v1s10n/1.png)

With a bit of digging it can be found that some static values of BMP are actually switched out in the hexdump
![Wikipedia binary structure](../assets/contents/tunn3l_v1s10n/2.png)
![Wikipedia binary structure example](../assets/contents/tunn3l_v1s10n/3.png)

Correcting the values 
![Target values](../assets/contents/tunn3l_v1s10n/4.png)

And... we can now open the file.
![Target values](../assets/contents/tunn3l_v1s10n/5.png)
However there seems to be a sizing issue

Find the resolution
```bash
achu@air ~ % file /Users/achu/Downloads/tunn3l_v1s10n.bmp 
/Users/achu/Downloads/tunn3l_v1s10n.bmp: PC bitmap, Windows 3.x format, 1134 x 306 x 24, image size 2893400, resolution 5669 x 5669 px/m, cbSize 2893454, bits offset 54
achu@air ~ % 
```
![Height and width values](../assets/contents/tunn3l_v1s10n/6.png)
Height and Width comes after the value we assigned `28` to.

Bitmap template tool in Hexfiend was a great aid
![Found a cool feature 1](../assets/contents/tunn3l_v1s10n/7.png)
![Found a cool feature 2](../assets/contents/tunn3l_v1s10n/8.png)

We edit those to a higher value (specfically set height to `900`, ie, `0x384`)
So we edit the value from `32 01` to `84 03`, and we finally get the flag!!!
![Found a cool feature 2](../assets/contents/tunn3l_v1s10n/9.png)

The flag is `picoCTF{qu1t3_a_v13w_2020}`.


# m00nwalk
We get a `wav` file, and from the gist of the following hint
> How did pictures from the moon landing get sent back to Earth? 

It can be assumed that this audio is some protocol for relaying images from moon.
![Wikipedia Research](../assets/contents/m00nwalk/1.png)
From the wikipedia research, it is clear that `SSTV` was used for capturing moments of moon walk.
I found this [SSTV decoder on github](https://github.com/colaclanth/sstv)
(It uses deprecated practices but it seems to be what we want)
```bash
~/SandBox$ git clone https://github.com/colaclanth/sstv.git
Cloning into 'sstv'...
remote: Enumerating objects: 221, done.
remote: Counting objects: 100% (59/59), done.
remote: Compressing objects: 100% (10/10), done.
remote: Total 221 (delta 51), reused 49 (delta 49), pack-reused 162 (from 1)
Receiving objects: 100% (221/221), 1.01 MiB | 5.58 MiB/s, done.
Resolving deltas: 100% (139/139), done.
~/SandBox/sstv$ python setup.py install
running install
/nix/store/wblvmd5y7izx0z10d1w7ga7zc4apjxmb-python3.11-setuptools-75.1.1/lib/python3.11/site-packages/setuptools/_distutils/cmd.py:66: SetuptoolsDeprecationWarning: setup.py install is deprecated.
!!

        ********************************************************************************
        Please avoid running ``setup.py`` directly.
        Instead, use pypa/build, pypa/installer or other
        standards-based tools.

        See https://blog.ganssle.io/articles/2021/10/setup-py-deprecated.html for details.
        ********************************************************************************

!!
  self.initialize_options()
/nix/store/wblvmd5y7izx0z10d1w7ga7zc4apjxmb-python3.11-setuptools-75.1.1/lib/python3.11/site-packages/setuptools/_distutils/cmd.py:66: EasyInstallDeprecationWarning: easy_install command is deprecated.
!!

        ********************************************************************************
        Please avoid running ``setup.py`` and ``easy_install``.
        Instead, use pypa/build, pypa/installer or other
        standards-based tools.

        See https://github.com/pypa/setuptools/issues/917 for details.
        ********************************************************************************

!!
  self.initialize_options()
running bdist_egg
running egg_info
creating sstv.egg-info
writing sstv.egg-info/PKG-INFO
writing dependency_links to sstv.egg-info/dependency_links.txt
writing entry points to sstv.egg-info/entry_points.txt
writing requirements to sstv.egg-info/requires.txt
writing top-level names to sstv.egg-info/top_level.txt
writing manifest file 'sstv.egg-info/SOURCES.txt'
reading manifest file 'sstv.egg-info/SOURCES.txt'
adding license file 'LICENSE'
writing manifest file 'sstv.egg-info/SOURCES.txt'
installing library code to build/bdist.linux-x86_64/egg
running install_lib
running build_py
creating build/lib/sstv
copying sstv/__init__.py -> build/lib/sstv
copying sstv/__main__.py -> build/lib/sstv
copying sstv/command.py -> build/lib/sstv
copying sstv/common.py -> build/lib/sstv
copying sstv/decode.py -> build/lib/sstv
copying sstv/spec.py -> build/lib/sstv
creating build/bdist.linux-x86_64/egg
creating build/bdist.linux-x86_64/egg/sstv
copying build/lib/sstv/__init__.py -> build/bdist.linux-x86_64/egg/sstv
copying build/lib/sstv/__main__.py -> build/bdist.linux-x86_64/egg/sstv
copying build/lib/sstv/command.py -> build/bdist.linux-x86_64/egg/sstv
copying build/lib/sstv/common.py -> build/bdist.linux-x86_64/egg/sstv
copying build/lib/sstv/decode.py -> build/bdist.linux-x86_64/egg/sstv
copying build/lib/sstv/spec.py -> build/bdist.linux-x86_64/egg/sstv
byte-compiling build/bdist.linux-x86_64/egg/sstv/__init__.py to __init__.cpython-311.pyc
byte-compiling build/bdist.linux-x86_64/egg/sstv/__main__.py to __main__.cpython-311.pyc
byte-compiling build/bdist.linux-x86_64/egg/sstv/command.py to command.cpython-311.pyc
byte-compiling build/bdist.linux-x86_64/egg/sstv/common.py to common.cpython-311.pyc
byte-compiling build/bdist.linux-x86_64/egg/sstv/decode.py to decode.cpython-311.pyc
byte-compiling build/bdist.linux-x86_64/egg/sstv/spec.py to spec.cpython-311.pyc
creating build/bdist.linux-x86_64/egg/EGG-INFO
copying sstv.egg-info/PKG-INFO -> build/bdist.linux-x86_64/egg/EGG-INFO
copying sstv.egg-info/SOURCES.txt -> build/bdist.linux-x86_64/egg/EGG-INFO
copying sstv.egg-info/dependency_links.txt -> build/bdist.linux-x86_64/egg/EGG-INFO
copying sstv.egg-info/entry_points.txt -> build/bdist.linux-x86_64/egg/EGG-INFO
copying sstv.egg-info/requires.txt -> build/bdist.linux-x86_64/egg/EGG-INFO
copying sstv.egg-info/top_level.txt -> build/bdist.linux-x86_64/egg/EGG-INFO
zip_safe flag not set; analyzing archive contents...
creating dist
creating 'dist/sstv-0.1-py3.11.egg' and adding 'build/bdist.linux-x86_64/egg' to it
removing 'build/bdist.linux-x86_64/egg' (and everything under it)
Processing sstv-0.1-py3.11.egg
Copying sstv-0.1-py3.11.egg to /home/runner/SandBox/.pythonlibs/lib/python3.11/site-packages
Adding sstv 0.1 to easy-install.pth file
Installing sstv script to /home/runner/SandBox/.pythonlibs/bin

Installed /home/runner/SandBox/.pythonlibs/lib/python3.11/site-packages/sstv-0.1-py3.11.egg
Processing dependencies for sstv==0.1
Searching for scipy
Reading https://pypi.org/simple/scipy/
Downloading https://files.pythonhosted.org/packages/0b/d8/036b4f4cada6c2e4af38ea2a7d4e7b24ddeb368265bf1775e25de8d959f0/scipy-1.15.0rc1-cp311-cp311-manylinux_2_17_x86_64.manylinux2014_x86_64.whl#sha256=fe65030d576d1d1426ed3c97e09a5365086a68c751a0cc79ad5690a8fc652462
Best match: scipy 1.15.0rc1
Processing scipy-1.15.0rc1-cp311-cp311-manylinux_2_17_x86_64.manylinux2014_x86_64.whl
Installing scipy-1.15.0rc1-cp311-cp311-manylinux_2_17_x86_64.manylinux2014_x86_64.whl to /home/runner/SandBox/.pythonlibs/lib/python3.11/site-packages
Adding scipy 1.15.0rc1 to easy-install.pth file
detected new path './sstv-0.1-py3.11.egg'

Installed /home/runner/SandBox/.pythonlibs/lib/python3.11/site-packages/scipy-1.15.0rc1-py3.11-linux-x86_64.egg
Searching for PySoundFile
Reading https://pypi.org/simple/PySoundFile/
Downloading https://files.pythonhosted.org/packages/2a/b3/0b871e5fd31b9a8e54b4ee359384e705a1ca1e2870706d2f081dc7cc1693/PySoundFile-0.9.0.post1-py2.py3-none-any.whl#sha256=db14f84f4af1910f54766cf0c0f19d52414fa80aa0e11cb338b5614946f39947
Best match: PySoundFile 0.9.0.post1
Processing PySoundFile-0.9.0.post1-py2.py3-none-any.whl
Installing PySoundFile-0.9.0.post1-py2.py3-none-any.whl to /home/runner/SandBox/.pythonlibs/lib/python3.11/site-packages
Adding PySoundFile 0.9.0.post1 to easy-install.pth file
detected new path './scipy-1.15.0rc1-py3.11-linux-x86_64.egg'

Installed /home/runner/SandBox/.pythonlibs/lib/python3.11/site-packages/PySoundFile-0.9.0.post1-py3.11.egg
Searching for Pillow
Reading https://pypi.org/simple/Pillow/
Downloading https://files.pythonhosted.org/packages/a9/9b/8a8c4d07d77447b7457164b861d18f5a31ae6418ef5c07f6f878fa09039a/pillow-11.0.0-cp311-cp311-manylinux_2_17_x86_64.manylinux2014_x86_64.whl#sha256=6f4dba50cfa56f910241eb7f883c20f1e7b1d8f7d91c750cd0b318bad443f4d5
Best match: pillow 11.0.0
Processing pillow-11.0.0-cp311-cp311-manylinux_2_17_x86_64.manylinux2014_x86_64.whl
Installing pillow-11.0.0-cp311-cp311-manylinux_2_17_x86_64.manylinux2014_x86_64.whl to /home/runner/SandBox/.pythonlibs/lib/python3.11/site-packages
Adding pillow 11.0.0 to easy-install.pth file
detected new path './PySoundFile-0.9.0.post1-py3.11.egg'

Installed /home/runner/SandBox/.pythonlibs/lib/python3.11/site-packages/pillow-11.0.0-py3.11-linux-x86_64.egg
Searching for numpy
Reading https://pypi.org/simple/numpy/
Downloading https://files.pythonhosted.org/packages/f2/d4/f999444e86986f3533e7151c272bd8186c55dda554284def18557e013a2a/numpy-2.2.1-cp311-cp311-manylinux_2_17_x86_64.manylinux2014_x86_64.whl#sha256=38efc1e56b73cc9b182fe55e56e63b044dd26a72128fd2fbd502f75555d92591
Best match: numpy 2.2.1
Processing numpy-2.2.1-cp311-cp311-manylinux_2_17_x86_64.manylinux2014_x86_64.whl
Installing numpy-2.2.1-cp311-cp311-manylinux_2_17_x86_64.manylinux2014_x86_64.whl to /home/runner/SandBox/.pythonlibs/lib/python3.11/site-packages
Adding numpy 2.2.1 to easy-install.pth file
detected new path './pillow-11.0.0-py3.11-linux-x86_64.egg'
Installing f2py script to /home/runner/SandBox/.pythonlibs/bin
Installing numpy-config script to /home/runner/SandBox/.pythonlibs/bin

Installed /home/runner/SandBox/.pythonlibs/lib/python3.11/site-packages/numpy-2.2.1-py3.11-linux-x86_64.egg
Searching for cffi>=0.6
Reading https://pypi.org/simple/cffi/
Downloading https://files.pythonhosted.org/packages/ff/6b/d45873c5e0242196f042d555526f92aa9e0c32355a1be1ff8c27f077fd37/cffi-1.17.1-cp311-cp311-manylinux_2_17_x86_64.manylinux2014_x86_64.whl#sha256=610faea79c43e44c71e1ec53a554553fa22321b65fae24889706c0a84d4ad86d
Best match: cffi 1.17.1
Processing cffi-1.17.1-cp311-cp311-manylinux_2_17_x86_64.manylinux2014_x86_64.whl
Installing cffi-1.17.1-cp311-cp311-manylinux_2_17_x86_64.manylinux2014_x86_64.whl to /home/runner/SandBox/.pythonlibs/lib/python3.11/site-packages
Adding cffi 1.17.1 to easy-install.pth file
detected new path './numpy-2.2.1-py3.11-linux-x86_64.egg'

Installed /home/runner/SandBox/.pythonlibs/lib/python3.11/site-packages/cffi-1.17.1-py3.11-linux-x86_64.egg
Searching for pycparser
Reading https://pypi.org/simple/pycparser/
Downloading https://files.pythonhosted.org/packages/13/a3/a812df4e2dd5696d1f351d58b8fe16a405b234ad2886a0dab9183fb78109/pycparser-2.22-py3-none-any.whl#sha256=c3702b6d3dd8c7abc1afa565d7e63d53a1d0bd86cdc24edd75470f4de499cfcc
Best match: pycparser 2.22
Processing pycparser-2.22-py3-none-any.whl
Installing pycparser-2.22-py3-none-any.whl to /home/runner/SandBox/.pythonlibs/lib/python3.11/site-packages
Adding pycparser 2.22 to easy-install.pth file
detected new path './cffi-1.17.1-py3.11-linux-x86_64.egg'

Installed /home/runner/SandBox/.pythonlibs/lib/python3.11/site-packages/pycparser-2.22-py3.11.egg
Finished processing dependencies for sstv==0.1
~/SandBox/sstv$ cd ..
~/SandBox$ sstv -d message.wav -o result.png
[sstv] Searching for calibration header... Found!    
[sstv] Detected SSTV mode Scottie 1
[sstv] Decoding image...   [##########################################################] 100%
[sstv] Drawing image data...
[sstv] ...Done!
~/SandBox$ 
```
This produces the result that we want:
![Result](../assets/contents/m00nwalk/2.png)
The flag is `picoCTF{beep_boop_im_in_space}`
