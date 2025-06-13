<h1>Brainpan 1</h1>

<h1>"Reverse engineer a Windows executable, find a buffer overflow and exploit it on a Linux machine."</h1>

Bom, já temos a dica para começar com o desafio...  
Vamos começar fazendo um scan do alvo para verificar as portas e serviços rodando.

![Captura de Tela (39)](https://github.com/user-attachments/assets/960ad5e3-b6bd-4803-99bc-f20cfeb8649d)

![Captura de Tela (40)](https://github.com/user-attachments/assets/da5d3fb0-52a9-4ab0-93ea-5a2ed82f03aa)

Fazendo um scan mais aprofundado, vemos na porta 10000 um servidor HTTP em Python, e na porta 9999 temos uma resposta de bytes.  
Damos uma olhada no site HTTP e não encontramos aparentemente nada demais.

![Captura de Tela (41)](https://github.com/user-attachments/assets/454bc5f7-a39a-412c-b159-db895c52ea7c)

Já na porta 9999 parece ter um app rodando.

![Captura de Tela (42)](https://github.com/user-attachments/assets/05a033ce-8f12-470e-b60e-7e8f85cd6d5a)

Confirmamos isso criando uma conexão com o NetCat. Recebemos a resposta do servidor — um app que pede uma senha.

![Captura de Tela (44)](https://github.com/user-attachments/assets/7df9cbe4-3a3d-4c1c-8004-d054792211ea)

Após um brute force de diretórios, é possível achar o diretório `/bin` no site HTTP, e ali encontramos a aplicação na qual faremos a engenharia reversa.

![Captura de Tela (43)](https://github.com/user-attachments/assets/74ed410c-720a-4305-b6bd-c688225e267b)

Ao ser um executável para Windows, vamos copiá-lo para nossa máquina Windows e começar a brincadeira.

![Captura de Tela (45)](https://github.com/user-attachments/assets/87c90de8-f954-4d70-956b-630a7a3c452b)

Analisando ele no IDA, vemos algumas strings interessantes.

![Captura de Tela (49)](https://github.com/user-attachments/assets/aeef5396-3714-4fcc-887c-3dd214e57995)

No pseudocode não achamos nada demais, mas podemos verificar como a app cria a conexão:

![Captura de Tela (48)](https://github.com/user-attachments/assets/a733df53-f949-4cca-9f65-5a2414a79c87)

Testando a string `"shitstorm"`, confirmamos que é a senha correta, mas ao inseri-la, nada acontece.

![Captura de Tela (55)](https://github.com/user-attachments/assets/90003995-5770-441c-8be8-ba1bdf443d80)

Segundo o enunciado do desafio, teremos que praticar um Buffer Overflow no alvo, então vamos partir para uma análise dinâmica.

Vamos criar um pattern para identificar após quantos bytes atingimos o endereço de retorno. Para isso utilizei o:  
- https://wiremask.eu/tools/buffer-overflow-pattern-generator/

![Captura de Tela (50)](https://github.com/user-attachments/assets/5991aa79-1d53-4c09-998d-280baa243d8d)

No screenshot acima, podemos ver o breakpoint setado na hora que é chamada a função de comparação de string, que vai comparar os valores que estão no topo da stack (nosso input e a string "shitstorm").

![Captura de Tela (51)](https://github.com/user-attachments/assets/0c3ddbc0-5dcd-4c7e-835b-a40ca6f15843)

Agora sim o endereço de retorno foi sobrescrito com o valor `35724134`.

![Captura de Tela (52)](https://github.com/user-attachments/assets/d9bd3e3d-271a-4b7d-a4c7-5dcff8b63593)

Vemos que após 524 bytes atingimos o endereço de retorno.

Vamos mandar então 524 `'A'` + `'BBBB'` (RET) + `'CCCCCCCC'` (que seria nosso shellcode) para ver como está a pilha na hora do retorno.

![Captura de Tela (56)](https://github.com/user-attachments/assets/c89f6f5f-f617-4e34-a637-408b78c2bf75)

Legal! Agora temos no ESP o nosso RET e vemos que o nosso shellcode estaria na stack no endereço `005FF910`.

Agora vamos ver no Immunity Debugger junto com o Mona se o executável tem algum tipo de proteção e quais DLLs carrega.

![Captura de Tela (57)](https://github.com/user-attachments/assets/5965bf0f-6c16-428a-b3e1-af7b15b6a51a)

Conferimos que o único módulo sem nenhum tipo de proteção é o próprio executável.

Para chamar o shellcode na pilha, temos que procurar por uma instrução `CALL ESP` ou `JMP ESP`.

Procurando por comandos similares no x64Dbg, achamos um `JMP ESP` no endereço `311712F3`, que será então o nosso RET.



Vamos agora criar um shellcode com o msfvenom para ter um reverse shell no alvo, que é uma máquina Linux.

![Captura de Tela (46)](https://github.com/user-attachments/assets/c7cc227f-b133-4043-a27a-b07b7aa2ca13)

Já com nosso shellcode, vamos criar nosso exploit em Python. Vamos precisar criar uma conexão com o alvo utilizando a biblioteca socket.

![Captura de Tela (61)](https://github.com/user-attachments/assets/ad5fa143-8585-4a6e-94f5-8e98cc97d537)

A estrutura do nosso código fica assim: 524 'A' para encher o espaço de memória + endereço de retorno (JMP ESP) + 16 NOPs + nosso shellcode + \n (enter)

![Captura de Tela (60)](https://github.com/user-attachments/assets/f2739dd8-54da-4501-b298-9be375bf0cb5)

Beleza! Conseguimos nossa shell. Agora, vamos escalar privilégios para o usuário root.

Usamos o comando `sudo -l` para listar os comandos que o usuário atual tem permissão para executar com sudo.

![Captura de Tela (62)](https://github.com/user-attachments/assets/a6be315a-d444-42fb-89de-d343f0f8984f)

Vemos que temos um binário que é executado como root. Ao ser chamado, ele mostra o seu uso. Vamos utilizar o manual de algum comando aleatório para depois chamar `!/bin/bash` e conseguir nossa shell de root.

![Captura de Tela (64)](https://github.com/user-attachments/assets/143c6115-e762-4171-87be-a87f4cc595e3)
