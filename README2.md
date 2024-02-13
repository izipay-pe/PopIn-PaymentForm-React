# PopIn-PaymentForm-React

## Índice

- [1. Introducción](#1-introducción)
- [2. Requisitos previos](#2-requisitos-previos)
- [3. Despliegue](#3-despliegue)
- [4. Datos de conexión](#4-datos-de-conexión)
- [5. Transacción de prueba](#5-transacción-de-prueba)
- [6. Implementación de la IPN](#6-implementación-de-la-ipn)
- [7. Personalización](#7-personalización)
- [8. Consideraciones](#8-consideraciones)

## 1. Introducción

En este manual podrás encontrar una guía paso a paso para configurar un proyecto de **[REACT]** con la pasarela de pagos de IZIPAY. Te proporcionaremos instrucciones detalladas y credenciales de prueba para la instalación y configuración del proyecto, permitiéndote trabajar y experimentar de manera segura en tu propio entorno local.
Este manual está diseñado para ayudarte a comprender el flujo de la integración de la pasarela para ayudarte a aprovechar al máximo tu proyecto y facilitar tu experiencia de desarrollo.

<p align="center">
  <img src="https://github.com/izipay-pe/Imagenes/blob/main/formulario_popin/formulario_popin.png" alt="Formulario" width="350"/>
</p>

<a name="Requisitos_Previos"></a>

## 2. Requisitos previos

- Comprender el flujo de comunicación de la pasarela. [Información Aquí](https://secure.micuentaweb.pe/doc/es-PE/rest/V4.0/javascript/guide/start.html)
- Extraer credenciales del Back Office Vendedor. [Guía Aquí](https://github.com/izipay-pe/obtener-credenciales-de-conexion)
- Debe instalar la [versión de LTS node.js](https://nodejs.org/en/).
- Para este proyecto utilizamos la herramienta Visual Studio Code.
  > [!NOTE]
  > Tener en cuenta que, para que el desarrollo de tu proyecto, eres libre de emplear tus herramientas preferidas.

## 3. Despliegue

- Ingrese a la carpeta raiz del proyecto.

- Agregue la dependencia **embedded-form-glue** o instale todas las dependencias que necesita el proyecto:

```sh
npm install --save @lyracom/embedded-form-glue
ó
npm install
```

- Ejecútelo y pruébelo:

```sh
npm start
```

ver el resultado en http://localhost:3000/

## 4. Datos de conexión

**Nota**: Reemplace **[CHANGE_ME]** con sus credenciales de `API REST` extraídas desde el Back Office Vendedor, ver [Requisitos Previos](#Requisitos_Previos).

- Editar en `public/index.html` en la sección HEAD.

```javascript
<!-- tema y plugins. debe cargarse en la sección HEAD -->
<link rel="stylesheet"
href="~~CHANGE_ME_ENDPOINT~~/static/js/krypton-client/V4.0/ext/classic-reset.css">
<script
    src="~~CHANGE_ME_ENDPOINT~~/static/js/krypton-client/V4.0/ext/classic.js">
</script>
```

- Edite el componente predeterminado `src/App.js`, con el siguiente codigo si quiere interactuar con el formulario de pago, con un endpoint propio.

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

### 4.1.- Verificación de hash de pago

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

## 5. Transacción de prueba

Antes de poner en marcha su pasarela de pago en un entorno de producción, es esencial realizar pruebas para garantizar su correcto funcionamiento.

Puede intentar realizar una transacción utilizando una tarjeta de prueba con la barra de herramientas de depuración (en la parte inferior de la página).

<p align="center">
  <img src="https://i.postimg.cc/3xXChGp2/tarjetas-prueba.png" alt="Formulario"/>
</p>

- También puede encontrar tarjetas de prueba en el siguiente enlace. [Tarjetas de prueba](https://secure.micuentaweb.pe/doc/es-PE/rest/V4.0/api/kb/test_cards.html)

## 6. Implementación de la IPN

> [!IMPORTANT]
> Es recomendable implementar la IPN para comunicar el resultado de la solicitud de pago al servidor del comercio.

La IPN es una notificación de servidor a servidor (servidor de Izipay hacia el servidor del comercio) que facilita información en tiempo real y de manera automática cuando se produce un evento, por ejemplo, al registrar una transacción.
Los datos transmitidos en la IPN se reciben y analizan mediante un script que el vendedor habrá desarrollado en su servidor.

- Ver manual de implementación de la IPN. [Aquí](https://secure.micuentaweb.pe/doc/es-PE/rest/V4.0/kb/payment_done.html)
- Vea el ejemplo de la respuesta IPN con PHP. [Aquí](https://github.com/izipay-pe/Redirect-PaymentForm-IpnT1-PHP)
- Vea el ejemplo de la respuesta IPN con NODE.JS. [Aquí](https://github.com/izipay-pe/Response-PaymentFormT1-Ipn)

## 7. Personalización

Si deseas aplicar cambios específicos en la apariencia de la pasarela de pago, puedes lograrlo mediante la modificación de código CSS. En este enlace [Código CSS - Incrustado](https://github.com/izipay-pe/Server-Personalization-PopIn) podrá encontrar nuestro script para un formulario incrustado.

## 8. Consideraciones

Para obtener más información, echa un vistazo a:

- [Formulario incrustado: prueba rápida](https://secure.micuentaweb.pe/doc/es-PE/rest/V4.0/javascript/quick_start_js.html)
- [Primeros pasos: pago simple](https://secure.micuentaweb.pe/doc/es-PE/rest/V4.0/javascript/guide/start.html)
- [Servicios web - referencia de la API REST](https://secure.micuentaweb.pe/doc/es-PE/rest/V4.0/api/reference.html)
