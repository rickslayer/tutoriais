# Tutoriais
Tutoriais simples e de grande Ajuda
# Índice
* [WordPress](#wordpress)
    * [Ajax com Javascript puro no Wordpress](#ajax-com-javascript-puro-no-wordpress)
* [JavaScript X Jquery](#javascript-x-jquery)
* [Linux](#linux)

## Wordpress
## Ajax com Javascript puro no Wordpress
Quando falamos de requisões Ajax no Wordpress, a maioria dos exemplos que encontramos é feito via Jquery.
A ideia desse tutorial é fazer somente com Javascript puro, sem depender do jQuery. **Vamos lá!!**

```objc 
<?php
/*Dentro do arquivo functions.php que fica dentro do tema do seu Wordpress, 
crie uma função que o ajax irá chamar.*/
function nome_da_acao()
{
    //recebendo os valores via POST
    $post = $_POST;
    
    echo json_encode($post);
    //o Wordpress Exige um die no final da execução
    die();
    
}
/*actions para liberar a requisão Ajax no Wordpress*/
add_action('wp_ajax_nopriv_nome_da_acao', 'nome_da_acao');
add_action('wp_ajax_nome_da_acao', 'nome_da_acao');

/*Note que as actions tem que conter o nome exato da função 'wp_ajax_nopriv_{nome_da_funcao}' 
e 'wp_ajax_{nome_da_funcao}'*/
```
Agora vamos para parte do front no arquivo que será o requisitador da função **nome_da_funcao()**
* Exemplo:
```objc
[...]
<button id="btn_acao">Ação</button>
//simulando a chamada da funcao via AJAX no click de um botão
<script>

    //seleciona o elemento html nesse caso será um botão
    let botao = document.getElementById("btn_acao");
    
    //cria um evento de clique para o botão
    botao.addEventListener("click", function(){
    
       //os parametros que quero passar pro PHP via POST
       //Note que o parametro action é obrigatório e deve levar o nome da funcão PHP que você quer chamar
        var param =  'action=nome_da_acao';
        
        //cria um objeto XMLHttpRequest
        var xhttp = new XMLHttpRequest();
           
        /*O Wordpress utiliza uma classe auxiliar chamada wp_ajax.php
        é obrigatório passar o caminho dessa classe para a URL do AJAX*/
        xhttp.open("POST", '/wp-admin/wp_ajax.php', true);
        
        xhttp.setRequestHeader('Content-Type', 'application/x-www-form-urlencoded');
        xhttp.onreadystatechange = function()
        {
            if (this.readyState == 4 && this.status == 200) {
                    //se tudo der certo, retorna a resposta do PHP
                    console.log(this.response);
               }
        };

         //envia a requisição tratando o encode da URI
         xhttp.send(encodeURI(param));

        });
    });

</script>
```
**Existem algumas práticas de segurança ao utilizar AJAX no WordPress. Pesquise e compartilhe**

## JavaScript X Jquery

## Linux
   
# Contribuidor
  Dúvidas ou algum erro? corrija ou mande um e-mail
 
* [Linkedin](https://www.linkedin.com/in/paulorick/)
* [GitHub](https://github.com/rickslayer/)
* [Email](mailto:paulo@actio.net.br)

