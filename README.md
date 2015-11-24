###Scripts for recent hotspot paper

####Step 1: Convert MS files to LDhat inputs
>Use 'MS2LDhat.pl' from https://github.com/lstevison/vcf-conversion-tools/blob/master/MS2LDhat.pl

####Step2: Run LDhat 
>Sites input denoted as $sites and Locs input denoted as $locs; Base filename indicated as $base

```
interval -seq $sites -loc $locs -lk lk_n30_t0.001 -its 60000000 -bpen 5 -samp 40000
stat -input rates.txt -burn 500 -loc $locs
```

>Use 'first_column.pl' from https://github.com/lstevison/great-ape-recombination/blob/master/first_column.pl to examine convergence summary

```
./first_column.pl 500 rates.txt bounds.txt consum.$base.txt
```

####Step3: Extract endpoints of LDhat results

```sh
#print res output into map BED file to create intervals, convert from kb to bp format
awk 'NR>2 && OFS="\t" {if(v)print "chr1",v*1000,$1*1000,rate;v=$1;rate=$2}' res.$base.txt >$base_map.BED

#extract last line information to print to map BED file
next_last_coord=$(tail -n2 $base.ldhat.locs | head -1 | awk '{print $1*1000}')
final_coord=$(tail -n1 $base.ldhat.locs | awk '{print $1*1000}')
final_rate=$(tail -n1 res.$base.txt | awk '{print $2}')

#print last line to map BED file
printf "chr1\t$next_last_coord\t$final_coord\t$final_rate\n" >>$base_map.BED
```

####Step4: Break up region into 1kb bins for subsequent averaging
>Use 'make_bins.pl' from https://github.com/lstevison/great-ape-recombination/blob/master/make_bins.pl

```sh
#get first coord and print with final coord from above
part1=$(head -1 $base_map.BED | awk 'OFS="\t" {print $1,$2}')
printf "$part1\t$final_coord\n" >$base.BED
sed -i 's/chr//g' $base.BED
./make_bins.pl $base.BED 1000 1kb
```

####Step5: Get recombination rate averages across 1kb bins from LDhat map
>Use 'rate_at_hotspots.pl' from https://github.com/lstevison/great-ape-recombination/blob/master/rate_at_hotspots.pl

```sh
#input map file from Step3, 1kb bins from Step4, output name, column with coordinates, column with rates, and coordinate format
./rate_at_hotspots.pl $base_map.BED $base.BED.binned_by_1kb.txt $base_averaged.txt 0 3 bp
```
