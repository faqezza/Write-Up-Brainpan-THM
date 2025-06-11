![Captura de Tela (49)](https://github.com/user-attachments/assets/b95e607a-eb9c-48e0-93de-b7ca2ed46381)<h1>Brainpan 1</h1>

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

Segundo o enunciado do desafio teremos que praticar um Buffer Overflow no alvo, entao vamos partir para uma analise dinamica.





