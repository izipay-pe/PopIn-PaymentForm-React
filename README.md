# Popin Payment Form React

<p align="justify">Ejemplo de un formulario Popin con REACT y la biblioteca de embedded-form-glue, este ejemplo te servira como guia para poder ejecutar el formulario de pago de Izipay dentro de cualquier proyecto que utilice la libreria de REACT. React te ayuda a crear interfaces de usuario interactivas de forma sencilla. Diseña vistas simples para cada estado en tu aplicación, y React se encargará de actualizar y renderizar de manera eficiente los componentes correctos cuando los datos cambien..<p>

<p align="center">
  <img src="https://github.com/izipay-pe/Imagenes/blob/main/formulario_popin/Imagen-Formulario-Popin.png?raw=true" alt="Popin" width="250"/>
</p>

## Este ejemplo es solo una guía para poder realizar la integración de la pasarela de pagos, puede realizar las modificaciones necesarias para su proyecto.

<a name="Requisitos_Previos"></a>

## Requisitos Previos

* Acceso al Back Office Vendedor (BOV) y Claves de autenticación. [Guía Aquí](https://github.com/izipay-pe/obtener-credenciales-de-conexion)
* Debe instalar la [versión de LTS node.js](https://nodejs.org/es/).

## 1.- Crear el proyecto 
* Descargar el proyecto .zip ingresando [aquí](https://github.com/izipay-pe/Popin-PaymentForm-React/archive/refs/heads/main.zip) ó clonarlo con git.
```sh
git clone https://github.com/izipay-pe/Popin-PaymentForm-React.git
``` 

* Ingrese a la carpeta raiz del proyecto.

* Agregue la dependencia **embedded-form-glue** o instale todas las dependencias que necesita el proyecto:

```sh
npm install --save @lyracom/embedded-form-glue
ó
npm install
```

* Ejecútelo y pruébelo:
```sh
npm start
```

ver el resultado en http://localhost:3000/

## 2.- Agregar el formulario de pago
**Nota**: Reemplace **[CHANGE_ME]** con sus credenciales de `API REST` extraídas desde el Back Office Vendedor, ver [Requisitos Previos](#Requisitos_Previos).

* Editar en `public/index.html` en la sección HEAD.

```javascript
<!-- tema y plugins. debe cargarse en la sección HEAD -->
<link rel="stylesheet"
href="~~CHANGE_ME_ENDPOINT~~/static/js/krypton-client/V4.0/ext/classic-reset.css">
<script
    src="~~CHANGE_ME_ENDPOINT~~/static/js/krypton-client/V4.0/ext/classic.js">
</script>
```

* Edite el componente predeterminado `src/App.js`, con el siguiente codigo si quiere interactuar con el formulario de pago, con un endpoint propio.

```javascript
import { useState } from 'react';
import KRGlue from '@lyracom/embedded-form-glue';
import axios from 'axios';
import PaymentForm from './components/PaymentForm';

function App() {

  const [isShow, setIsShow] = useState(false);
  const [isValid, setIsValid] = useState(true);
  const [amount, setAmount] = useState("");

  const publicKey = "~~CHANGE_ME_PUBLIC_KEY~~";
  const endPoint = "~~CHANGE_ME_ENDPOINT~~";
  const formToken = "~~DEMO_TOKEN-TO-BE-REPLACED~~";
  const server = "http://your_server.com";
  
  const payment = ()=>{...}

  const getFormToken = (monto, publicKey, domain) => {
    const dataPayment = {
        amount: monto*100,
        currency: "PEN",
        customer:{
          email: "example@gmail.com"
        },
        orderId: "pedido-0"
    }
    KRGlue.loadLibrary(domain,publicKey)
    .then(({KR}) => KR.setFormConfig({
    formToken: formToken
    }))
    .then(({ KR }) => KR.onSubmit(validatePayment) )
    .then(({ KR }) => KR.attachForm("#form") )
    .then(({ KR, result }) => KR.showForm(result.formId))
    .catch(err=>console.log(err))
  }

  const validatePayment = (resp) => {...}

  return (
    <div className='container'>
        <main>
            <div className="py-5 text-center">      
                ...   
            </div>

            <div className="row g-5">
                <div className="col-md-5 col-lg-4 order-md-last">
                    <div className="d-flex justify-content-center">
                        <div id="myDIV" className="formulario" style={{display: isShow?"block":"none" }}>
                            <div id="form">
                                {/* Formulario de pago POPIN */}
                                <PaymentForm popin={true} />
                            </div> 
                        </div>                         
                    </div>                    
                    <hr className="my-4"/>
                </div>

                <div className="col-md-7 col-lg-8">          
                    <form className="needs-validation">
                        ...   
                        <button onClick={payment} className="w-100 btn btn-primary btn-lg" type="button">Finalizar con el Pago</button>
                    </form>
                </div>
            </div>
        </main>
    </div>
  );
}
export default App;
```

## 2.1.- Verificación de hash de pago

El hash de pago debe validarse en el lado del servidor para evitar la exposición de su clave hash personal.

En el lado del servidor:

```js
const express = require('express')
const hmacSHA256 = require('crypto-js/hmac-sha256')
const Hex = require('crypto-js/enc-hex')
const app = express()
(...)
// válida los datos de pago dados (hash)
app.post('/validatePayment', (req, res) => {
  const answer = req.body.clientAnswer
  const hash = req.body.hash
  const answerHash = Hex.stringify(
    hmacSHA256(JSON.stringify(answer), 'CHANGE_ME: HMAC SHA256 KEY')
  )
  if (hash === answerHash) res.status(200).send('Valid payment')
  else res.status(500).send('Payment hash mismatch')
})
(...)
```

Del lado del cliente:

```js
import { useState } from 'react';
import KRGlue from '@lyracom/embedded-form-glue';
import axios from 'axios';
import PaymentForm from './components/PaymentForm';

function App() {

  const [isShow, setIsShow] = useState(false);
  const [isValid, setIsValid] = useState(true);
  const [amount, setAmount] = useState("");

  const publicKey = "~~CHANGE_ME_ENDPOINT~~";
  const endPoint = "~~CHANGE_ME_ENDPOINT~~";
  const formToken = "~~CHANGE_ME_ENDPOINT~~";
  const server = "http://localhost:3000";
  
  const payment = ()=>{...}
  const getFormToken = (monto, publicKey, domain) => {
    const dataPayment = {
        amount: monto*100,
        currency: "USD",
        customer:{
          email: "example@gmail.com"
        },
        orderId: "pedido-0"
    }
    KRGlue.loadLibrary(domain,publicKey)
    .then(({KR}) => KR.setFormConfig({
    formToken: formToken
    }))
    .then(({ KR }) => KR.onSubmit(validatePayment) )
    .then(({ KR }) => KR.attachForm("#form") )
    .then(({ KR, result }) => KR.showForm(result.formId))
    .catch(err=>console.log(err))

  }

  const validatePayment = (resp) => {
        axios.post(`${server}/validatePayment`, resp)
    .then(({data}) => {
      if (data==="Valid Payment"){
        setIsShow(false);
        alert("Pago Satisfactorio");
        
      }else{
        alert("Pago Inválido");
      }
    })
    return false;
  }
(...)
```

## 3.- Transacción de prueba

El formulario de pago está listo, puede intentar realizar una transacción utilizando una tarjeta de prueba con la barra de herramientas de depuración (en la parte inferior de la página).

 ![debug](https://github.com/izipay-pe/Embedded-PaymentForm-T1.Net/blob/main/images/tarjetasprueba2.png)

Si intenta pagar, tendrá el siguiente error: **CLIENT_100: Demo form, see the documentation**.
Es porque el **formToken** es inválido, está definido usando **KR.setFormConfig** y está configurado en **DEMO-TOKEN-TO-BE-REPLACED**.

you have to create a **formToken** before displaying the payment form using Charge/CreatePayment web-service.

Para obtener más información, eche un vistazo a:

- [Formulario incrustado: prueba rápida](https://secure.micuentaweb.pe/doc/es-PE/rest/V4.0/javascript/quick_start_js.html)
- [Primeros pasos: pago simple](https://secure.micuentaweb.pe/doc/es-PE/rest/V4.0/javascript/guide/start.html)
- [Servicios web - referencia de la API REST](https://secure.micuentaweb.pe/doc/es-PE/rest/V4.0/api/reference.html)

**NOTA**

1.- Paso de la tienda al modo PRODUCTION 

     Modifique su implementación para utilizar el incrustado en producción:
     * la contraseña de producción,
     * clave pública de producción,
     * la clave HMAC-SHA-256 de producción para calcular la firma contenida en el campo kr-hash.
     
2.- No tengo una cuenta activa con Izipay. [Suscribete Aquí](https://online.izipay.pe/comprar/cliente)

   | CARACTERÍSTICAS | VALOR |
   | ------------- | ------------- |
   | Usuario de prueba  | 89289758  |
   | Contraseña de prueba  | testpassword_7vAtvN49E8Ad6e6ihMqIOvOHC6QV5YKmIXgxisMm0V7Eq  |
   | Clave pública de prueba  | 89289758:testpublickey_TxzPjl9xKlhM0a6tfSVNilcLTOUZ0ndsTogGTByPUATcE  |
   | Clave HMAC SHA256 de prueba  | fva7JZ2vSY7MhRuOPamu6U5HlpabAoEf8VmFHQupspnXB  |
   | URL de base  | https://api.micuentaweb.pe |
   | URL para el cliente JavaScript | https://api.micuentaweb.pe/static/js/krypton-client/V4.0/stable/kr-payment-form.min.js  |

## 4.- Gestionar la notificacion de fin de pago (IPN)

IPN son las siglas de Instant Payment Notification (URL de notificación instantánea, en inglés). Al crear una transacción o cambiar su estado, nuestros servidores emitirán una IPN que llamará a una URL de notificación en sus servidores. Esto le permitirá estar informado en tiempo real de los cambios realizados en una transacción.

Las IPN son la única manera de recibir notificaciones en los casos siguientes:

* La conexión a Internet del comprador se ha cortado.
* El comprador cierra su navegador durante el pago.
* Se ha rechazado una transacción.
* El comprador no ha terminado su pago antes de la expiración de su sesión de pago.

Por lo tanto, es obligatorio integrar las IPN.

   <p align="center">
     <img src="https://github.com/izipay-pe/Embedded-PaymentForm-.Net/raw/main/images/IPN-imagen.png?raw=true" alt="Formulario"/>
   </p>  

* Ver manual de implementacion de la IPN [Aquí](https://secure.micuentaweb.pe/doc/es-PE/rest/V4.0/kb/payment_done.html)

* Ver el ejemplo de la respuesta IPN con PHP [Aquí](https://github.com/izipay-pe/Redirect-PaymentForm-IpnT1-PHP)

* Ver el ejemplo de la respuesta IPN con NODE.JS [Aquí](https://github.com/izipay-pe/Response-PaymentFormT1-Ipn)
