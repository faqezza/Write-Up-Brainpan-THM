<h1>Brainpan 1</h1>

<h1>"Reverse engineer a Windows executable, find a buffer overflow and exploit it on a Linux machine."</h1>



Bom ja temos a dica para começar com o desafio...
Vamos começar fazendo um scan do alvo para verificar as portas e serviços rodando.

![Captura de Tela (39)](https://github.com/user-attachments/assets/960ad5e3-b6bd-4803-99bc-f20cfeb8649d)


![Captura de Tela (40)](https://github.com/user-attachments/assets/da5d3fb0-52a9-4ab0-93ea-5a2ed82f03aa)


Fazendo um scan mais aporifundado vemos na porta 10000 um servidor http em Python e na porta 9999 temos uma resposta de bytes.
Damos uma olhada no site http e nao encontramos aparentemente nada de mais.


![Captura de Tela (41)](https://github.com/user-attachments/assets/454bc5f7-a39a-412c-b159-db895c52ea7c).




Ja na porta 9999 parece ter um app rodando.



![Captura de Tela (42)](https://github.com/user-attachments/assets/05a033ce-8f12-470e-b60e-7e8f85cd6d5a).



Confirmamos isso criando uma conexao com o NetCat recevemos a resposta do servidor, um app que pede uma senha.


![Captura de Tela (44)](https://github.com/user-attachments/assets/7df9cbe4-3a3d-4c1c-8004-d054792211ea).

Apos um brute force de diretorios é possivel achar o diretorio /bin no site http e ali encontramos a aplicacao na qual faremos a engenieria reversa.


![Captura de Tela (43)](https://github.com/user-attachments/assets/74ed410c-720a-4305-b6bd-c688225e267b).

Ao ser um executavel para windows vamos copialo para nossa maquina windows e comecar a brincadeira.






![Captura de Tela (45)](https://github.com/user-attachments/assets/87c90de8-f954-4d70-956b-630a7a3c452b)





Analisando ele no IDA vemos algumas strings interesantes. 

![Captura de Tela (49)](https://github.com/user-attachments/assets/aeef5396-3714-4fcc-887c-3dd214e57995)


no pseudo code nao achamos nada demais mas podemos verificar como a app cria a conexao

![Captura de Tela (48)](https://github.com/user-attachments/assets/a733df53-f949-4cca-9f65-5a2414a79c87)

Testanto a string "shitstorm" confirmamos que é a senha correta mas ao inserila nada acontece.

![Captura de Tela (55)](https://github.com/user-attachments/assets/90003995-5770-441c-8be8-ba1bdf443d80)


Segundo o enunciado do desafio teremos que praticar um Buffer Overflow no alvo, entao vamos partir para uma analise dinamica.

Vamos criar um pattern para identificar apos quantos bytes atingimos o endereco de retorno para isso utilizei o:
- https://wiremask.eu/tools/buffer-overflow-pattern-generator/





![Captura de Tela (50)](https://github.com/user-attachments/assets/5991aa79-1d53-4c09-998d-280baa243d8d)
No screenshot acima podemos ver o breakpoint setado na hora que é chamada a funcao string comparar que vai comparar os valores que estao no topo da stack(nosso imput e a string "shitstorm")




![Captura de Tela (51)](https://github.com/user-attachments/assets/0c3ddbc0-5dcd-4c7e-835b-a40ca6f15843)

Agora sim o endereco de retorno foi sobre escrito com o valor 35724134.



![Captura de Tela (52)](https://github.com/user-attachments/assets/d9bd3e3d-271a-4b7d-a4c7-5dcff8b63593)  


Vemos que apos 524 Bytes atingimos o endereco de retorno.



Vamos mandar entao 524 'A' + 'BBBB'(RET) + 'CCCCCCCC'(que seria nosso shellcode) para ver como esta a pilha na hora do retorno.


![Captura de Tela (56)](https://github.com/user-attachments/assets/c89f6f5f-f617-4e34-a637-408b78c2bf75)




Legal agora temos no ESP nosso RET e vemos que o nosso shellcode estaria na stack no endereço 005FF910.

Agora vamos ver no Inmunity Debuger junto com o Mona se o executavel tem algum tipo de protecao e quais dll's carrega.

![Captura de Tela (57)](https://github.com/user-attachments/assets/5965bf0f-6c16-428a-b3e1-af7b15b6a51a)


conferimos que o unico midulo sem nehum tipo de protecao é o propio executavel.

Vamos partir para a escrita do exploit.
