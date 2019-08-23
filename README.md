# Bot


<?php
/*
  Incluimos las librerias que usaremos en el proceso,
  en este caso, haremos uso de formity2
*/
require_once('formity2.php');

/*
  Creamos los formularios que usaremos, a cada uno le asiganos un nombre
  con el cual lo invocaremos en adelante.
*/
$re = Formity::getInstance('Ficha');
$re->addField('nombre', 'input:text');
$re->addField('descripcion', 'input:text');
$re->addField('Numero', 'select')->setOptions(array('1' => 'Uno', '2' => 'Dos'));


/*
  Configuramos el Bot con el archivo de fileMind,
  donde se almaneca datos de la conversación.
*/
Bot::config($message->fileMind);

/*
  Registramos Formity al Bot, de tal forma que tenemos registrado
  los eventos para cada caso, como: exito o cancelación. Lo haremos ingresando el nombre
  con el cual creamos el formulario
*/
Bot::registerFormity('Ficha', function($queue, $re) {
  $queue->reply(json_encode($re->getData()));
  $queue->goQueue(1);
}, function($queue) {
  $queue->reply("Se ha cancelado el formulario");
  $queue->goQueue(1);
});


/*
  Creamos las colas del Bot, en el cual separaremos las escuchas
*/
$queue = Bot::createQueue(1);

/*
  Registramos las escuchas de la cola creada, estas pueden ser expresiones regulares,
  textos literales o una función que revuelva true o false
*/ 
$queue->hears('iniciar', function ($queue) {
  /*
    De esta forma podemos brindar una respuesta inmediata al mensaje recibido
  */
  $queue->reply("Iniciamos el formulario:");
  /*
    De esta forma invocamos al formulario, de esta forma sigue la secuencia del formulario 
  */
  $queue->replyFormity('Ficha');
});
$queue->hears('opciones', function ($queue) {
  $queue->reply("mis opciones");
});
$queue->hears(function($n) {
  return strpos($n->rtexto, 'hola') !== false;
}, function($queue) {
  $queue->reply('Hola! ¿En qué podemos ayudarte?');
});

/*
  Definimos cual es la cola que escuchará en un inicio
*/
Bot::listen(1, $message);

/*
  Se elimina los formularios usados en el proceso,
  de lo contrario se creará conflictos en servicio.
*/
Formity::delete('Ficha');

