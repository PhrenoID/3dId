# Review of 3D_ID software

[The outline for the review](https://github.com/PhrenoID/3dId/blob/master/outline.md)

## First obtain the software and the data from http://www.3d-id.org/download-3d-id
```
wget http://www.3d-id.org/download-3d-id/3d_id.jar
jar xf 3d_id.jar
#here is the data
cd j3d_id/j3d_id/data
```


Get data from the .mdt file into R-compatble .csv

```
perl -e '$o="";while(<STDIN>){chop();$o=$1 if m/^OBJ "([^"]*)"/;if (m/ENDOBJ/){for $k (keys %d){print "$o;$k;$d{$k}\n"};%d=();$o="";$n=0};if ($o ne ""){if (m/^USERTEXT "([^"]*)" "([^"]*)"/){$d{$1} = $2;}else{ if (m/\s*P "([^"]*)" (.*)$/){ $d{($n++).";$1"}=$2} }}i}' < 3d_id.mdt  > 3d_id.kv

cat 3d_id.kv | perl -e 'open A,"labs";@l=();while(<A>){chop();push @l, $_;};while(<STDIN>){chop();@x=split(/;/);if($x[1]=~/^[0-9]/){$x[3]=~s/^\s*//;$x[3]=~s/\s*$//;$x[3]=~s/\s+/ /g;$x[3]=~s/\s+/;/g;$v{$x[0]}{$x[1]}=$x[3];}else{$v{$x[0]}{$x[1]}=$x[2];};};for $o (keys %v){print "$o"; for $a (@l){print ";$v{$o}{$a}"}print"\n"}' > 3d_id.csv
```

Let's see what it is
```
R --no-save
x=read.table("3d_id.csv1",sep=';',quote="",comment.char="");
l=read.table("labs");
labs=as.character(l[1:5,1]);
labs=c("id",labs,paste(rep(c("x","y","z"),35),rep(1:35,rep(3,35)),sep="."));
names(x)=labs;
#make NAs
for (i in 7:111) x[x[,i]==-999,i] = NA;


#now all observationas are ready
summary(x[,2:8])
    COLLECTION        GROUP                 POP     
 DONATED   :181            :716                 :635  
 TERRY     :174   white    :173   Oloriz        :130  
 OLORIZ    :130   black    : 53   black         :108  
 Morton    :124   Turkey   : 33   Hispanic_Chile: 74  
 UT_DONATED:118   Macedonia: 22   white         : 74  
           : 99   (Other)  : 74   Cuba_African  : 27  
 (Other)   :246   NA's     :  1   (Other)       : 24  
                            RACE           SEX           x.1         
                              :1001   male   :442   Min.   :-252.17  
 Hispanic_Guatamala           :  21   female :295   1st Qu.:  29.28  
 White                        :  15          :217   Median : 117.85  
 hispanic_panama              :   8   M      : 48   Mean   :  83.27  
 hispanic_peru                :   8   ?      : 43   3rd Qu.: 160.11  
 hispanic_panama_afroantillean:   6   F      :  9   Max.   : 332.60  
 (Other)                      :  13   (Other): 18   NA's   :15       

```

These are basic stats. The first component of the first variable is
all over the place, is the stuff rotation invariant?


```
table(is.na(apply(x[,7:111],1,sum)))
FALSE  TRUE 
111   961
```
only 111 observations are complete?



```
table(is.na(apply(x[,7:(7+3*5)],1,sum)))
FALSE  TRUE 
 1034    38 
```

first five (3d) measures are OK, though, lets look at them:

```
sel = is.na(apply(x[,7:(7+3*5)],1,sum));
ev = prcomp(x[!sel,7:(7+3*5)])$sdev;
cumsum(ev)/sum(ev)
[1] 0.3972115 0.6789811 0.8178958 0.8713678 0.9154606 0.9439236 0.9613881
```

Three principal components account for 82% of variance, thats out of 
the total of 15 variables that are predominantly not missing.

