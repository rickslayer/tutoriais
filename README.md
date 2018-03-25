# Tutoriais
Tutoriais simples e de grande Ajuda
# Índice
* [WordPress](#wordpress)
    * [Ajax com Javascript puro no Wordpress](#ajax-com-javascript-puro-no-wordpress)
    * [Envio de Email Sem Plugin no Wordpress](#envio-de-email-sem-plugin-no-wordpress)
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

## Envio de Email Sem Plugin no Wordpress
Criando uma classe de envio de e-mail no Wordpress sem auxilio de plugin

Criaremos uma classe para ser chamada na funcions.php do seu tema no Wordpress.
Também utilizaremos o [@PHPMailer](https://github.com/PHPMailer/PHPMailer) uma das classes mais utilizadas
no auxílio de envio de e-mail.
```objc
classe envia_email.controller.php
<?php
require($_SERVER['DOCUMENT_ROOT'].'/wp-load.php');

class envia_email_controller
{
    private $to;
    private $assunto;
    private $corpo;
    private $mensagem;
    function __construct()
    {
        
    }
    
    function Send()
    {
        date_default_timezone_set('America/Sao_Paulo');
        $install_path = $_SERVER['DOCUMENT_ROOT'];
        require_once($install_path. '/wp-includes/class-phpmailer.php');
        require_once($install_path. '/wp-includes/class-smtp.php');
        $result = new stdClass();   
         
        $mail = new PHPMailer();
        $mail->IsSMTP();
        $mail->CharSet='UTF-8';
        $mail->setFrom('from@email.com', 'Titulo do Remetente');
        //seta o assunto do e-mail
        $mail->Subject = $this->getAssunto();
    
        $mail->SMTPAuth = true;  
        $mail->SMTPSecure = 'ssl'; 
        $mail->SMTPAutoTLS = false;
        $mail->Host = 'host.com.br';
        $mail->Port = 465;
        $mail->Username = 'emailusuario@com.br';
        $mail->Password = suasenhadeemail;      
        $mail->SMTPDebug  = 0;
        $mail->addAddress( $this->getTo() );
        $mail->IsHTML(true);
        $mail->Body = $this->getCorpo();
        
        
        if(!$mail->send()){
                    $result->message  = "Erro ao enviar o e-mail";
                    $result->success  = false;
                    $result->error    = $mail->ErrorInfo;
                    echo json_encode($result);
             
                } else {
                    $result->message = $this->getMensagem();
                    $result->success = true;
                    echo json_encode($result);
                }
        //Configurações do Google Gmail
         /* $mail->SMTPAuth = true;  
        $mail->SMTPSecure = 'tls'; 
        $mail->SMTPAutoTLS = false;
        $mail->Host = 'smtp.gmail.com';
        $mail->Port = 587;
        $mail->Username = 'teste@gmail.com';
        $mail->SMTPDebug  = 0;*/
    }
     
    /**
     * @return mixed
     */
    public function getTo()
    {
        return $this->to;
    }
      /**
     * @return mixed
     */
    public function getMensagem()
    {
        return $this->mensagem;
    }

    /**
     * @return mixed
     */
    public function getAssunto()
    {
        return $this->assunto;
    }

    /**
     * @return mixed
     */
    public function getCorpo()
    {
        return $this->corpo;
    }

     /**
     * @param mixed $to
     */
    public function setTo($to)
    {
        $this->to = $to;
    }
      /**
     * @param mixed $mensagem
     */
    public function setMensagem($mensagem)
    {
        $this->mensagem = $mensagem;
    }

    /**
     * @param mixed $assunto
     */
    public function setAssunto($assunto)
    {
        $this->assunto = $assunto;
    }

    /**
     * @param mixed $corpo
     */
    public function setCorpo($corpo)
    {
        $this->corpo = $corpo;
    }
}

```
Agora no arquivo functions.php do nosso tema vamos chamar nossa classe
```objc
no arquivo functions.php
<?php
            function enviar_email()
            {
                 require_once('envia_email_controller.php');
                 
                 $envia_email = new envia_email_controller(); //cria o objeto da classe
                 $envia_email->setTo('emaildodestinatario@mail.com');
                 $envia_email->setAssunto('Assunto do email');
                 $envia_email->setMensagem('Pode se passar algum retorno');
                 $envia_email->setCorpo(<html></html>); //passar o corpo do email em html
                 $envia_email->Send(); //envio do email
            }
```
A função pode ser chamada no front por AJAX , por exemplo, ou em qualquer rotina no back do Wordpress.

## JavaScript X Jquery
* Selecionar elemento HTML
```objc
<script>
      //jQuery
      $("#id_do_elemento") ou $(".classe_do_elemento")
      //JavaScript
      document.getElementById("id_do_elemento") ou document.getElementsByClassName("classe_do_elemento")
</script>
``` 

## Linux
   
# Contribuidor
  Dúvidas ou algum erro? corrija ou mande um e-mail
 
* [Linkedin](https://www.linkedin.com/in/paulorick/)
* [GitHub](https://github.com/rickslayer/)
* [Email](mailto:paulo@actio.net.br)

