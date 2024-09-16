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
[Полный отчет OST001_R1_001.fastq.gz](http://localhost:8888/doc/tree/qc/OST001_R1_001_fastqc.html)
[Полный отчет OST001_R2_001.fastq.gz](http://localhost:8888/doc/tree/qc/OST001_R2_001_fastqc.html)


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
[Полный отчет output_R1_paired.fq.gz](http://localhost:8888/doc/tree/Trimmomatic-0.39/fastqc_trimmed/output_R1_paired_fastqc.html)
[Полный отчет output_R2_paired.fq.gz](http://localhost:8888/doc/tree/Trimmomatic-0.39/fastqc_trimmed/output_R2_paired_fastqc.html)
```
mkdir fastqc_trimmed
fastqc -o fastqc_trimmed output_R1_paired.fq.gz output_R2_paired.fq.gz
```
<img width="1434" alt="Снимок экрана 2024-09-16 в 21 47 28" src="https://github.com/user-attachments/assets/505ce4ef-666f-4cba-ad5e-d8c088e8416f">
<img width="1431" alt="Снимок экрана 2024-09-16 в 21 47 48" src="https://github.com/user-attachments/assets/cdf4fa66-87dd-46b4-b360-56b1b7b8454e">

