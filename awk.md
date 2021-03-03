# 

to pull a specified region from a fasta file with one line 

```
# chrII:363400-364449

awk 'BEGIN{RS=">";FS="\n"}NR>1{if ($1~/chrII/)print ">"$0}'  genome.fasta | awk 'BEGIN{RS =">"; FS="\n"}NR>1{seq="";for (i=2;i<=NF;i++) seq=seq""$i; print ">"$1"\n"substr(seq,363400,364449-363400+1)}'
```
