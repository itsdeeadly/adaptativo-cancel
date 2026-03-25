# adaptativo-cancel

Copie e cole no console o seguinte comando para desativar o **bloqueio de copia e cola do site** *"Adaptativo SESI"*

```
const s=document.createElement('style');s.textContent=`*{user-select:text!important;-webkit-user-select:text!important;-moz-user-select:text!important;-ms-user-select:text!important}.exam-protected,.exam-protected *{-webkit-touch-callout:text!important}`;document.head.appendChild(s);document.querySelectorAll('*').forEach(e=>{e.oncopy=e.oncut=e.onpaste=null});document.addEventListener('copy',e=>e.stopImmediatePropagation(),true);document.querySelectorAll('.Toastify__toast').forEach(t=>t.remove());
```
