# snp_calling
## Ссылка на референс 
[Homo_sapiens.GRCh38.dna_sm.primary_assembly.fa](https://ftp.ensembl.org/pub/release-112/fasta/homo_sapiens/dna/Homo_sapiens.GRCh38.dna_sm.primary_assembly.fa.gz)
Основной задачей стоит ознакомление с необходимым софтом и проведение анализа индивидуальных мутаций. Работа выполнялась в JupyterLab, этапы установки софта на локальную машину намеренно пропущены. 

# Шаг №1 "Проверка качества чтений и тримминг по необходимости"
## Используемые инструменты: FastQC, Trimmomatic 

После скачивания ридов прогнала их через FastQC, чтобы проверить их качество 

'mkdir qc
fastqc -o qc --noextract data/OST001_R1_001.fastq.gz
fastqc -o qc --noextract data/OST001_R2_001.fastq.gz'
