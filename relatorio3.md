---
author:
- Thiago Cunha Ferreira e Eduardo Freire de Carvalho Lima
title: 'EP4: MAC0422 - Sistemas Operacionais'
---

Introdução
==========

Este relatório descreve o que foi feito para o quarto EP
(Exercício-Programa) da matéria *MAC0422 - Sistemas Operacionais*, onde
tivemos que fazer modificações no sistema operacional para:

-   Acrescentar um novo tipo de arquivo, o *arquivo temporário*.

**IMPORTANTE: DURANTE A INICIALIZAÇÃO DO MINIX, NÃO SELECIONE AS OPÇÕES
1 OU 2 DE INICIALIZAÇÃO. ESPERE OU USE A OPÇÃO 3.**

Trechos de códigos modificados estão marcados entre uma longa sequência
do símbolo \# em forma de comentários, geralmente ocupando uma linha
inteira. Deve-se ter em conta que estamos modificando o SO a partir de
imagens dos EP's passados, então pode haver trechos no código fonte
marcados com essa delimitação de outras etapas. Por isso, considere como
arquivos modificados para esse EP os listados neste relatório e, se um
mesmo arquivo foi modificado nas duas fases, deve haver uma demarcação
de qual trecho foi modificado quando.

Lista de arquivos modificados/adicionados
=========================================

Listamos aqui os diversos arquivos que apresentam modificações. Também
citamos arquivos criados prescindido pelo simbolo \"+\".

-   \+ $/usr/src/lib/posix/\_open_tmp.c$

    Define a chamada de sistema *open\_tmp()*. Os modos existentes e
    parte de sua implementação foram baesadas no código da função
    *fopen()* da biblioteca padrão do C *stdio*. Aqui traduzimos os
    modos de acesso e tentamos acessar um arquivo existente igual ao
    caminho dado. Se não for possível, criamos um novo arquivo. No
    final, retornamos um FD (*FILE DESCRIPTOR*).

-   $/usr/src/include/minix/const.h$

    Adicionado macro que representa um tipo temporário, I\_TEMPORARY,
    com valor octal de *140000*.

-   $/usr/src/lib/posix/Makefile.in$

    Adicionado arquivo \_open\_tmp.c para que possa ser gerado um novo
    Makefile (ao executar o comando *make Makefile*), esse podendo ser
    utilizado por outras ferramentas de compilação do SO.

-   $/usr/src/include/fcntl.h$

    Adicionado protótipo da chamada de sistema. Apesar ela poder ser
    usada mesmo sem incluí-la no projeto, possibilita a remoção das
    mensagens de *warning* do compilador *CC* em relação a função.

-   $/usr/src/include/minix/callnr.h$

    Adicionado o número da chamada de sistema para comunicação com o
    *File System* (em $/usr/src/servers/fs/table.c$), número 70, código
    OPEN\_TMP.

-   $/usr/src/servers/fs/proto.h$

    Adicionado protótipo da função *do\_open\_tmp()* em
    $/usr/src/servers/pm/misc.c$.

-   $/usr/src/servers/fs/table.c$

    Adicionado mapeamento entre número da chamada de sistema e sua
    implementação no *File System*.

-   $/usr/src/servers/fs/open.c$

    Aqui é feita a parte principal da nova chamada de sistema, onde
    abrimos/criamos arquivos. Assim, temos uma função nova,
    *do\_open\_tmp()*, que faz a mesma coisa que a função *do\_open()*,
    mas chama a função que realmente faz o trabalho, *common\_open()*,
    com um novo parâmetro final que identifica se temos a intenção de
    abrir/criar um arquivo temporário. Após isso, retornamos o FD para o
    usuário.

    Nesse mesmo arquivo, foi necessária uma mudança na função
    *do\_close()*, que implementa a chamada de sistema *close()*, uma
    vez que precisávamos que a entrada da tabela *filp* corresponde ao
    FD passado para a função não fosse tornado nulo, visto que
    precisamos dessa entrada durante o processo de terminação do
    processo para eliminar o arquivo.

-   $/usr/src/servers/fs/misc.c$

    Aqui foram feitas mudanças na função *free\_proc()*, que serve para
    liberar as estruturas do *File System* quando um processo acaba
    (através da chamada de sistema *exit()*). Após fecharmos os arquivos
    relacionados ao processo (através de várias chamadas ao
    *do\_close()*), verificamos, para cada um deles, se são arquivos
    temporários. Se forem, removemo-os do sistema. Para tanto, não
    pudemos ter acesso a campos mais pertinentes relacionados a cada
    arquivo, somente o seu i-node. Assim há uma limitação do número de
    arquivos temporários que serão excluídos.

-   $[/usr/src/servers/fs/]misc.c, link.c, read.c$

    Aqui só adicionamos a checagem de tipos para o tipo temporário de
    arquivos, assim permitindo que eles possam fazer coisas parecidas
    com os arquivos normais.

Informações adicionais
----------------------

A compilação desse EP foi feito localmente em algumas partes e usando o
Makefile presente em $/usr/src$.

Certas funcionalidades dos arquivos temporários podem não estar
incompletas, visto a proximidade com a data de entrega, junto da
relativa complexidade encontrada no *File System*.
