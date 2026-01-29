# Guia de criação de QuickBMS

Este guia cobre os conceitos fundamentais e os comandos essenciais do QuickBMS para escrever scripts de extração e reimportação.

## Ferramentas necessárias:
- QuickBMS — Luigi Auriemma: https://aluigi.altervista.org/quickbms.htm
- HXD — Freeware Hex Editor and Disk Editor: https://mh-nexus.de/en/hxd/

## Informações necessárias para criar um script:
**Para criar um script de QuickBMS precisamos de saber algumas informações como:**
- **Endianess**: Se é Little Endian ou Big Endian. Exemplo: uma int com valor: 1.  
**Little Endian**: Hex: 0x01000000
**Big Endian**:    Hex: 0x00000001.
- Entender a estrutura do arquivo.
- Entender a pouco de programação, hex editing.
- Se o ficheiro possui algum tipo de compressão.

### Tipo váriaveis:
- Tamanhos usados para ler valores dentro do ficheiro.

| QuickBMS: | Programação: | Tamanho: | Descrição: |
|:--------:|:-----------:|:---------:|:---------:|
| `Byte`     | `byte`   | 1 bytes | Valor inteiro de 8 bits, variando de 0x00 (0) a 0xFF (255). |
| `Short`     | `short`   | 2 bytes | Valores inteiro de 16 bits. |
| `Long`     | `int`   | 4 bytes | Valores inteiro de 32 bits. |
| `Float`     | `float`   | 4 bytes | Valores decimais de 32 bits. Ex: 0.01. |
| `string`     | `string`   | Sem tamanho fixo. | Cadeia de carateres. Um texto por exemplo. |

### Termos Hexadecimais:
- Os termos usados para mexer em ficheiros hexadecimais.

| Termo: | Descrição: |
|:--------:|:-----------:|
| **MagicID ou Assinatura**  | Assinatura do ficheiro, normalmente está sempre no inicio do ficheiro. | 
| **Endianness**  | É a ordem dos bytes que o ficheiro é escrito e lido. | 
| **Ponteiros**  | Um valor encontrado no ficheiro que aponta para um endereço. | 
| **Endereços ou Offset**  | Um endereço no ficheiro. | 


### Comandos básicos de scripts QuickBMS:
- `comtype (Compressão)`: Define a compressão.
- `endian big|little`: define ordem de bytes.
- `idstring "MAGIC"`: valida assinatura.
- `get VAR TYPE`: lê valores.
- `goto OFFSET`: navega.
- `savepos VAR`: guarda posição.
- `log NAME OFFSET SIZE`: extrai dados.
- `for/while/if`: controle de fluxo.
- `string VAR ...`: manipula texto.

Com todos os conceitos estabelecidos em cima passamos para prática, por exemplo o seguinte arquivo:

```
struct KypArchive //Estrutura do arquivo.
{
    string MagicID; //Assinatura do arquivo.
    int FileSize; //Tamanho do arquivo.
    int FileCount; //Total de ficheiros dentro do arquivo.
    int Padding; //Espaçamento  dentro do arquivo.

    List<KypFileEntry> Files; //Ficheiros dentro do arquivo.
}
```

```
struct KypFileEntry //Detalhes de cada ficheiro.
{
    int Offset; //Aponta onde o ficheiro se encontra
    int Length; //Tamanho do ficheiro.
    string Name //O nome desse ficheiro.
}
```

Seguindo as informações obtemos o seguinte script:

```
endian big //Seta o endianess do leitor.
//Inicialmente iniciamos por ler a header.
idstring "KYP\x00" //Lê a assinatura do arquivo.
get archiveSize long //Lê o tamanho total do arquivo que está armazenado 4 bytes.
get fileCount long //Lê a quantidade total de arquivos que está armazenado 4 bytes.
get null long //Lê o Padding: 0x00000000
//A seguir temos o KypFileEntry, começamos a extrair os arquivos via loop contado do zero até quantidade de ficheiros.
for i = 0 < fileCount
   get pointer long //Lê o endereço do ficheiro dentro arquivo
   get length long //Lê o Tamanhodo ficheiro dentro arquivo
   getdstring name 0x8 //Lê a string com o tamanho de 8 bytes que é o nome do ficheiro.
   log name pointer length //Ele assigna o nome, viaja até ao ponteiro, extrair o arquivo até ao tamanho registado.
next i //Continua o loop.
```
