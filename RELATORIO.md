# 📝 Relatório do Laboratório 2 - Chamadas de Sistema

---

## 1️⃣ Exercício 1a - Observação printf() vs 1b - write()

### 💻 Comandos executados:
```bash
strace -e write ./ex1a_printf
strace -e write ./ex1b_write
```

### 🔍 Análise

**1. Quantas syscalls write() cada programa gerou?**
- ex1a_printf: 8 syscalls
- ex1b_write: 7 syscalls

**2. Por que há diferença entre os dois métodos? Consulte o docs/printf_vs_write.md**
Há diferença, pois o printf é capaz de escrever textos para o usuário, formatar dados e é utilizado para casos que não seja necessário uma performance crítica.
E o write é capaz de escrever, além de texto, dados binários, comtrolar quando enviar os dados e possui um comportamento previsível.

**3. Qual método é mais previsível? Por quê você acha isso?**
O método mais previsível é o write, pois toda vez que ela é chamada resulta em uma syscall. Além disso, ao utilizar esse método, obtém-se controle total sobre o momento de envio dos dados. Dessa forma permitindo saber o que e quando as operacoes estao acontecendo.

## 2️⃣ Exercício 2 - Leitura de Arquivo

### 📊 Resultados da execução:
- File descriptor: 3
- Bytes lidos: 127

### 🔧 Comando strace:
```bash
strace -e openat,read,close ./ex2_leitura
```

### 🔍 Análise

**1. Qual file descriptor foi usado? Por que não começou em 0, 1 ou 2?**
O file descriptor utilizado foi o 3, pois é o próximo disponível após o processo de open. O sistema operacional cria automaticamente os file descriptors 0, 1 e 2, que correspondem, respectivamente, ao standard input, standard output e standard error. Esses são alocados antes da abertura do arquivo dados/teste1.txt, o que explica por que o próximo descriptor atribuído é o 3.

**2. Como você sabe que o arquivo foi lido completamente?**

O indicativo é que, além de exibir mensagens que o arquivo foi aberto e fechado, também é mostrada a quantidade de bytes lidos, o que mostra que o arquivo foi lido completamente.

**3. Por que verificar retorno de cada syscall?**

É essencial para verificar se todos os programas inseridos estao sendo executados corretamente e que não houve um erro de leitura do arquivo ou outros diveros erros em algum momento do processo.

## 3️⃣ Exercício 3 - Contador com Loop

### 📋 Resultados (BUFFER_SIZE = 64):
- Linhas: 25.
- Caracteres: 1300
- Chamadas read(): 21
- Tempo: 0.000140 segundos

### 🧪 Experimentos com buffer:

| Buffer Size | Chamadas read() | Tempo (s) |
|-------------|-----------------|-----------|
| 16          |       82        | 0.000093  |
| 64          |       21        | 0.000140  |
| 256         |        6        | 0.000042  |
| 1024        |        2        | 0.000048  |

### 🔍 Análise

**1. Como o tamanho do buffer afeta o número de syscalls?**
  Foi observado que, quanto maior o tamanho do buffer size, menos chamadas são necessárias para realizar a leitura. Isso ocorre porque buffers maiores são capazes de interpretar mais informação e precisam de menos camadas para completar o processo de leitura.

**2. Todas as chamadas read() retornaram BUFFER_SIZE bytes? Discorra brevemente sobre**

Nem todas as chamadas read() retornaram o BUFFER_SIZE. Isso acontece, pois no final da leitura do arquivo, nao havia espaco ou dados suficiente para preencher o buffer.

**3. Qual é a relação entre syscalls e performance?**
  A relação é que, quanto menos chamadas de sistema são feitas, melhor é a performance de um sistema, porque, além de as chamadas exigirem um processamento mais pesado, toda vez que ela é feita, o processador deve trocar para o modo kernel, o que consome muito tempo.

---

## 4️⃣ Exercício 4 - Cópia de Arquivo

### 📈 Resultados:
- Bytes copiados: 1364
- Operações: 6
- Tempo: 0.000231 segundos
- Throughput: 5766.37 KB/s

### ✅ Verificação:
```bash
diff dados/origem.txt dados/destino.txt
```
Resultado: [ 1364 ] Idênticos [ 0 ] Diferentes

### 🔍 Análise

**1. Por que devemos verificar que bytes_escritos == bytes_lidos?**
  Para confirmar que que o conteudo que se deseja ser copiado para outro arquivo não tenha sido perdido. E essencial para verificar a integridade dos dados.

**2. Que flags são essenciais no open() do destino?**
  Os flags essenciais são: O_WRONLY | O_CREAT | O_TRUNC, 0644.

**3. O número de reads e writes é igual? Por quê?**
A quantidade de reads é igual à de writes, porque as operações de leitura e escrita estão copiando o conteúdo do arquivo de origem para o de destino, bloco por bloco. Além disso, se executarmos o comando strace -T -e read,write ./ex4_copia, é possível observar que, nas chamadas de sistema para copiar o conteúdo e escrevê-lo, há a mesma quantidade de operações de read e write.

**4. Como você saberia se o disco ficou cheio?**
  O indicativo seria checar se a chamada write() do programa retorna -1. Isso significaria que o disco ficou cheio.
  
**5. O que acontece se esquecer de fechar os arquivos?**
  Os arquivos podem sofrer um tipo de corrupcao de arquivo, perda de dados, uso excessivo de recursos e podem ficar bloqueados para outros processos.
---

## 🎯 Análise Geral

### 📖 Conceitos Fundamentais

**1. Como as syscalls demonstram a transição usuário → kernel?**
  No sistema o uso de chamadas syscalls, como read e write, sao um dos comandos que o usuario possui para solicitar serviços do sistema operacional. Toda vez que um asyscall é executada, essa instruções interrompe o fluxo normal (modo usuario) e transfere o controle para o kernel.

**2. Qual é o seu entendimento sobre a importância dos file descriptors?**
  Os files descriptors sao importantes para realizar operações como read, write, close funcionarem de forma uniforme, pelo fato de tudo ser tratado como um arquivo, que seja uma conexao de rede, dispositivos entre outros.

**3. Discorra sobre a relação entre o tamanho do buffer e performance:**
  Foi observado que com os exercicios praticos que o buffer tem a funcao de armazernar bytes. E quanto maior era seu armazenamento para, tanto como transferencia de dados, quanto para copia de dados, sua performance aumentava.

### ⚡ Comparação de Performance
  Ao realizar o experimento com diferentes tamanhos de buffer, observou-se que buffers pequenos (64 e 256 bytes) não impactam significativamente o tempo de execução, mas com buffers maiores (1024 e 4096 bytes), o tempo melhora, sendo que o comando cp do sistema se mostrou mais eficiente que o programa 'ex4_copia'.

**Qual foi mais rápido?** 
  Na maior parte dos casos, o comando cp teve uma performance mais rapida em relacao ao programa 'ex4_copia'.

**Por que você acha que foi mais rápido?**
  A performance do comando cp foi mais rápida pelo fato de ser capaz de realizar cópias diretas entre arquivos, sem a necessidade de sincronização de dados; dessa forma, não sobrecarrega o sistema e leva menos tempo para realizar a operação.

## 📤 Entrega
Certifique-se de ter:
- [ ] Todos os códigos com TODOs completados
- [ ] Traces salvos em `traces/`
- [ ] Este relatório preenchido como `RELATORIO.md`

```bash
strace -e write -o traces/ex1a_trace.txt ./ex1a_printf
strace -e write -o traces/ex1b_trace.txt ./ex1b_write
strace -o traces/ex2_trace.txt ./ex2_leitura
strace -c -o traces/ex3_stats.txt ./ex3_contador
strace -o traces/ex4_trace.txt ./ex4_copia
```
# Bom trabalho!
