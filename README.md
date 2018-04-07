# Tutoriais
Tutoriais simples e de grande Ajuda
# Índice
* [WordPress](#wordpress)
    * [Ajax com Javascript puro no Wordpress](#ajax-com-javascript-puro-no-wordpress)
    * [Envio de Email Sem Plugin no Wordpress](#envio-de-email-sem-plugin-no-wordpress)
    * [Aplicar Evento Purchase do Facebook Pixel no Woocommerce](#aplicar-evento-purchase-do-facebook-pixel-no-woocommerce)
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

## Aplicar Evento Purchase do Facebook Pixel no Woocommerce
   Tive muita dificuldade com o evento purchase do [@Pixel](https://www.facebook.com/business/learn/facebook-ads-pixel) para adicionar no [@Woocommerce](https://github.com/woocommerce/).
   Não achei nenhum plugin que fizesse isso de forma correta. Criei na mão mesmo.
   Como o evento recebe um parametro dinâmico que é o preço, torna-se um pouco complicado para plugins resolver isto.
   Resolvi de forma simples. Veja:
   
   ```objc
   /**
   * Encontre o arquivo chamado thankyou.php dentro da pasta /wp-content/themes/seu-tema/woocommerce/checkout/thankyou.php
   **/
   //A página começa geralmente dessa forma
   <?php
/**
 * Thankyou page
 *
 * This template can be overridden by copying it to yourtheme/woocommerce/checkout/thankyou.php.
 *
 * HOWEVER, on occasion WooCommerce will need to update template files and you
 * (the theme developer) will need to copy the new files to your theme to
 * maintain compatibility. We try to do this as little as possible, but it does
 * happen. When this occurs the version of the template file will be bumped and
 * the readme will list any important changes.
 *
 * @see 	    https://docs.woocommerce.com/document/template-structure/
 * @author 		WooThemes
 * @package 	WooCommerce/Templates
 * @version     3.2.0
 */

if ( ! defined( 'ABSPATH' ) ) {
	exit;
}
?>
//Eu criei um elemento HTML para armazenar o valor do pedido utilizando o objeto 
$order do woocommerce que me trás todos os objetos do pedido
<input type="hidden" id="total_pedido" name="total_pedido" value="<?php echo $order->total;?>">

//Agora o script do pixel eu coloquei no final da última div da página ficando assim:
 //Pixel Facebook
!function(f,b,e,v,n,t,s)
	{if(f.fbq)return;n=f.fbq=function(){n.callMethod?
	n.callMethod.apply(n,arguments):n.queue.push(arguments)};
	if(!f._fbq)f._fbq=n;n.push=n;n.loaded=!0;n.version='2.0';
	n.queue=[];t=b.createElement(e);t.async=!0;
	t.src=v;s=b.getElementsByTagName(e)[0];
	s.parentNode.insertBefore(t,s)}(window,document,'script',
	'https://connect.facebook.net/en_US/fbevents.js');
	
   //capiturando o campo total_pedido e armazenando na variavel valor
	var valor= document.getElementById('total_pedido');
   
    //capiturando com JQuery
    // var valor = $("#total_pedido");
       
	fbq('init', 'id-do-seu-pixel'); 
	fbq('track', 'Purchase', {'value':valor.value,'currency':'BRL'});
   //com jquery
   //fbq('track', 'Purchase', {'value':$(valor).val(),'currency':'BRL'});

</script>
<noscript>
	<img height="1" width="1" 
	src="https://www.facebook.com/tr?id=ID-DO-SEU-PIXEL&ev=Purchase
	&noscript=1"/>
	</noscript>
   </div>
   ```
   **É isso**

## JavaScript X Jquery

* Tabuada com JavaScript Exemplo:
<code>https://codepen.io/rickslayer/pen/YavxYm</code>

* Carregar JSON via AJAX em um elemento Lista Javascript Puro:
<code>https://codepen.io/rickslayer/pen/vRrRNp</code>

* Carregar JSON via AJAX em um elemento Lista  com AngularJS:
<code>https://codepen.io/rickslayer/pen/ZxRxrW</code>

* Trabalhando com o Recaptcha do Google
<code>https://codepen.io/rickslayer/pen/YaROzZ</code> ou copie o código em <code>['/rickslayer/recaptcha'](https://github.com/rickslayer/rickslayer.github.io/blob/master/recaptcha/index.html)</code>

## Linux
   
# Contribuidor
  Dúvidas ou algum erro? corrija ou mande um e-mail
 
* [Linkedin](https://www.linkedin.com/in/paulorick/)
* [GitHub](https://github.com/rickslayer/)
* [Email](mailto:paulo@actio.net.br)

