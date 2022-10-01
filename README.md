# hse22_hw1
1) Создаем папку с ДЗ1 и добавляем ссылку на папку с данными:
```
ln -s /usr/share/data-minor-bioinf/assembly/
mkdir hw1
mv assembly/ hw1/assembly/
cd hw1/
```

2) Создаем случайные чтения:
```
seqtk sample -s418 assembly/oil_R1.fastq 5000000 > R1_paired_end.fastq
seqtk sample -s418 assembly/oil_R2.fastq 5000000 > R2_paired_end.fastq
seqtk sample -s418 assembly/oilMP_S4_L001_R1_001.fastq 1500000 > R1_mate_pairs.fastq
seqtk sample -s418 assembly/oilMP_S4_L001_R2_001.fastq 1500000 > R2_mate_pairs.fastq
```

3) Оцениваем качество чтений (fastQC и multiQC):
```
mkdir fastqc
ls *.fastq | xargs -P 4 -tI{} fastqc -o fastqc {}

mkdir multiqc
multiqc -o multiqc fastqc
```

4) Установим миниконду для дальнейшего анализа:
```
wget https://repo.anaconda.com/miniconda/Miniconda3-py39_4.12.0-Linux-x86_64.sh
bash Miniconda3-py39_4.12.0-Linux-x86_64.sh
```
Установим jupyter, пробросим порт и откроем вкладку jupyter lab на локальном компьютере.

5) С помощью программ platanus_trim и platanus_internal_trim подрезаем чтения по качеству и удаляем праймеры:
```
platanus_trim R1_paired_end.fastq R2_paired_end.fastq
platanus_internal_trim R1_mate_pairs.fastq R2_mate_pairs.fastq
```

удаляем исходные .fastq файлы:
```
rm *.fastq
```
6) С помощью программы fastQC и multiQC оцениваем качество подрезанных чтений и получаем по ним общую статистику:
```
mkdir trimmed_fastq
mv -v *trimmed trimmed_fastq/

mkdir trimmed_fastqc
ls trimmed_fastq/* | xargs -P 4 -tI{} fastqc -o trimmed_fastqc {}

mkdir trimmed_multiqc
multiqc -o trimmed_multiqc trimmed_fastqc
```

some pictures


7) Cобираем контиги из подрезанных чтений:
```
time platanus assemble -o Poil -f trimmed_fastq/R1_paired_end.fastq.trimmed trimmed_fastq/R2_paired_end.fastq.trimmed 2> assemble.log &
```

8) Собираем скаффолды:
```
time platanus scaffold -o Poil -c Poil_contig.fa -IP1 trimmed_fastq/R1_paired_end.fastq.trimmed trimmed_fastq/R2_paired_end.fastq.trimmed -OP2 trimmed_fastq/R1_mate_pairs.fastq.int_trimmed trimmed_fastq/R2_mate_pairs.fastq.int_trimmed 2> scaffold.log &
```

9) Анализ полученных контигов и скаффолдов:

picture

10) Создаем файл с самым длинным скаффолдом: 
```
echo scaffold1_len3834621_cov231 > scaffold_name.txt
seqtk subseq Poil_scaffold.fa scaffold_name.txt > longest_scaffold.fna
```

11) Собираем скаффолды с меньшим количеством пропусков:
```
time platanus gap_close -o Poil -c Poil_scaffold.fa -IP1 trimmed_fastq/R1_paired_end.fastq.trimmed  trimmed_fastq/R2_paired_end.fastq.trimmed -OP2 trimmed_fastq/R1_mate_pairs.fastq.int_trimmed trimmed_fastq/R2_mate_pairs.fastq.int_trimmed 2> gapclose.log &
```

12) Создаем файл с самым длинным скаффолдом с меньшим количеством пропусков:
```
echo scaffold1_cov231 > longest_scaffold_gapclosed.txt
seqtk subseq Poil_gapClosed.fa longest_scaffold_gapclosed.txt > longest_scaffold_gapclosed.fna
```

13) Сравним полученные данные о гэпах в самом большом скаффолде до и после использования platanus gap_close

picture

14) Повторяем все действия для меньшего количества чтений (5 млн => 1 млн, 1.5 млн => 300 тыс.)

picture
