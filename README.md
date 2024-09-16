# snp_calling
## Ссылка на референс 
[Homo_sapiens.GRCh38.dna_sm.primary_assembly.fa](https://ftp.ensembl.org/pub/release-112/fasta/homo_sapiens/dna/Homo_sapiens.GRCh38.dna_sm.primary_assembly.fa.gz)
Основной задачей стоит ознакомление с необходимым софтом и проведение анализа индивидуальных мутаций. Работа выполнялась в JupyterLab, этапы установки софта на локальную машину намеренно пропущены, чаще всего создавалась отдельная для инструментов среда. 

# Шаг №1 "Проверка качества чтений и тримминг по необходимости"
## Используемые инструменты: FastQC, Trimmomatic 

После скачивания ридов прогнала их через FastQC, чтобы проверить их качество 
```
mkdir qc
fastqc -o qc --noextract data/OST001_R1_001.fastq.gz
fastqc -o qc --noextract data/OST001_R2_001.fastq.gz
```
## Анализ чтений 
[Полные отчеты](https://drive.google.com/drive/folders/1tilYaXeUwdwjjifDh6jp4n9t8-JXbgMu?usp=sharing)



Результаты FastQC парно-концевых чтений представлены ниже. Основными параметрами являются "Per base sequence quality" и в данном случае "Adapter Content", так как присутствуют неудаленные адаптеры. Качество прочтения последних нуклеотидов, особенно для файла OST001_R2_001.fastq.gz, снижено, по сравнению с качеством прочтения рида в целом, а также обнаруживается неудаленный универсальный адаптер Illumina, поэтому потребуется обрезка и фильтрация чтений с помощью Trimmomatic.
<img width="1432" alt="Снимок экрана 2024-09-16 в 21 19 21" src="https://github.com/user-attachments/assets/659074e2-252b-424c-9b5f-efbea54203fe">
<img width="1433" alt="Снимок экрана 2024-09-16 в 21 21 19" src="https://github.com/user-attachments/assets/cfe41ed5-ac2f-4758-9406-985af9c03165">
<img width="1437" alt="Снимок экрана 2024-09-16 в 21 25 24" src="https://github.com/user-attachments/assets/3362b0b3-f28e-40ac-b333-6b2d342673dc">

## Обрезка и фильтрация чтений
Параметры следующие: 
1. ILLUMINACLIP:TruSeq3-PE-2.fa:2:30:10 -- обрезка универсального адаптера Illumina, последовательность в файле TruSeq3-PE-2.fa, остальное дефотлтные настройки
2. LEADING:20 -- обрезка начальных нуклеотидов с качеством меньше 20. Таких практически нет, значение бралось как нижняя граница желтого коридора FastQC
3. TRAILING:20 -- обрезка кончных нуклеотидов с качеством меньше 20. Значение бралось как нижняя граница желтого коридора FastQC
4. MINLEN:75 -- минимальная длина после обрезки, оставила достаточно низкий порог
```
cd Trimmomatic-0.39
java -jar trimmomatic-0.39.jar PE  OST001_R1_001.fastq.gz OST001_R2_001.fastq.gz output_R1_paired.fq.gz output_R1_unpaired.fq.gz output_R2_paired.fq.gz output_R2_unpaired.fq.gz ILLUMINACLIP:TruSeq3-PE-2.fa:2:30:10 LEADING:20 TRAILING:20 MINLEN:75
```
## Далее еще один прогон через FastQC, чтобы убедиться, что обрезка прошла как надо
Понадобятся только файлы с парными чтениями. В целом все в порядке, синяя линяя показывает среднее качество прочтения каждого нуклеотида в риде, в обоих файлах среднее качество в зеленом диапазоне. Длины ридов в диапазоне 144-152 bp.

[Полные отчеты](https://drive.google.com/drive/folders/1tilYaXeUwdwjjifDh6jp4n9t8-JXbgMu?usp=sharing)
```
mkdir fastqc_trimmed
fastqc -o fastqc_trimmed output_R1_paired.fq.gz output_R2_paired.fq.gz
```
<img width="1434" alt="Снимок экрана 2024-09-16 в 21 47 28" src="https://github.com/user-attachments/assets/505ce4ef-666f-4cba-ad5e-d8c088e8416f">
<img width="1431" alt="Снимок экрана 2024-09-16 в 21 47 48" src="https://github.com/user-attachments/assets/cdf4fa66-87dd-46b4-b360-56b1b7b8454e">

# Этап №2 Выравнивание на референс Homo_sapiens.GRCh38.dna_sm.primary_assembly.fa, сортировка и фильтрация
## BWA-MEM
Маппинг занял около 6 часов вместе с индексацией референсного генома. 
1. BWA-backtrack -- для ридов Illumina до 100 bp
2. BWA-SW -- для ридов Illumina от 70 bp до 1 Mbp
3. BWA-MEM -- для ридов Illumina от 70 bp до 1 Mbp, но быстрее и точнее, последняя версия. Беру это
```
conda activate bwa_env
bwa index -p refseq Homo_sapiens.GRCh38.dna_sm.primary_assembly.fa
bwa mem refseq Trimmomatic-0.39/output_R1_paired.fq.gz Trimmomatic-0.39/output_R2_paired.fq.gz > align_data2.sam
```
Получили файл align_data2.sam, который нужно сортировать, отметить дубликаты и перевести в формат .bam, чтобы он занимал меньше места
## Picard -- сортировка по координате и маркировка дубликатов  
```
java -jar picard.jar SortSam -I align_data2.sam -O align_data2_sorted.sam -SORT_ORDER coordinate -VALIDATION_STRINGENCY SILENT
java -jar picard.jar MarkDuplicates -I align_data2_sorted.sam -O sort_dedup2.bam -METRICS_FILE metrics.txt -ASSUME_SORTED true -VALIDATION_STRINGENCY SILENT
```
## Samtools -- индексрованный файл и краткая оценка происходящего
По краткой оценке происходящего с помощью samtools flagstat видим, что замапилось 99,89% всех ридов. Это очень хороший результат.
```
conda activate samtools_env
samtools index pi/sort_dedup2.bam
samtools flagstat pi/sort_dedup2.bam
```
> 5765206 + 0 in total (QC-passed reads + QC-failed reads)

> 5753102 + 0 primary

> 0 + 0 secondary

> 12104 + 0 supplementary

> 272028 + 0 duplicates

> 272028 + 0 primary duplicates

> 5758669 + 0 mapped (99.89% : N/A)

> 5746565 + 0 primary mapped (99.89% : N/A)

> 5753102 + 0 paired in sequencing

> 2876551 + 0 read1

> 2876551 + 0 read2

> 5715368 + 0 properly paired (99.34% : N/A)

> 5741488 + 0 with itself and mate mapped

> 5077 + 0 singletons (0.09% : N/A)

> 22598 + 0 with mate mapped to a different chr

> 16201 + 0 with mate mapped to a different chr (mapQ>=5)

