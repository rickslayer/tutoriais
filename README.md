# Tutoriais
Diversos tutoriais sobre diversos temas relacionados a programação web.
# Índice

* [Ajax com Javascript puro no Wordpress](#ajax-com-javascript-puro-no-wordpress)



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

//Note que as actions tem que conter o nome exato da função 'wp_ajax_nopriv_{nome_da_funcao}' e 'wp_ajax_{nome_da_funcao}'
````
* 
