# adaptativo-cancel

Copie e cole no console o seguinte comando para desativar o **bloqueio de copia e cola do site** *"Adaptativo SESI"*

```
(function() {
    // 1. Força seleção de texto com CSS
    const style = document.createElement('style');
    style.textContent = `
        * {
            user-select: text !important;
            -webkit-user-select: text !important;
            -moz-user-select: text !important;
            -ms-user-select: text !important;
        }
        .exam-protected, .exam-protected * {
            -webkit-touch-callout: text !important;
        }
        [contenteditable="false"] {
            contenteditable: true !important;
        }
    `;
    document.head.appendChild(style);

    // 2. Remove classe e atributos que restringem
    document.querySelectorAll('.exam-protected').forEach(el => el.classList.remove('exam-protected'));
    document.querySelectorAll('[contenteditable="false"], [draggable="true"]').forEach(el => {
        if (el.hasAttribute('contenteditable')) el.setAttribute('contenteditable', 'true');
        if (el.hasAttribute('draggable')) el.removeAttribute('draggable');
    });

    // 3. Remove atributos inline de eventos
    document.querySelectorAll('*').forEach(el => {
        el.removeAttribute('oncopy');
        el.removeAttribute('oncut');
        el.removeAttribute('onpaste');
        el.removeAttribute('onselectstart');
        el.removeAttribute('ondragstart');
        el.removeAttribute('oncontextmenu');
    });

    // 4. Remove listeners globais antigos
    window.oncopy = null;
    document.oncopy = null;

    // 5. Bloqueia ADIÇÃO de novos listeners de cópia
    const originalAddEventListener = EventTarget.prototype.addEventListener;
    EventTarget.prototype.addEventListener = function(type, listener, options) {
        const blockedTypes = ['copy', 'cut', 'paste', 'selectstart', 'dragstart'];
        if (blockedTypes.includes(type)) {
            console.log(`[desbloqueio] Listener de ${type} bloqueado.`);
            return;
        }
        return originalAddEventListener.call(this, type, listener, options);
    };

    // 6. Intercepta e impede a propagação dos eventos de cópia já existentes
    const stopCopy = (e) => {
        e.stopImmediatePropagation();
        // Não chama preventDefault() – assim o comportamento padrão da cópia ocorre normalmente
    };
    document.addEventListener('copy', stopCopy, true);
    document.addEventListener('cut', stopCopy, true);
    document.addEventListener('paste', stopCopy, true);

    // 7. Remove os toasts que aparecem ao tentar copiar
    const removeToasts = () => {
        document.querySelectorAll('.Toastify__toast').forEach(t => t.remove());
    };
    removeToasts();
    // Observa novos toasts e remove imediatamente
    const toastObserver = new MutationObserver(() => removeToasts());
    toastObserver.observe(document.body, { childList: true, subtree: true });

    // 8. Observa mudanças no DOM para manter as alterações
    const domObserver = new MutationObserver(mutations => {
        mutations.forEach(mutation => {
            mutation.addedNodes.forEach(node => {
                if (node.nodeType === 1) {
                    if (node.classList && node.classList.contains('exam-protected')) {
                        node.classList.remove('exam-protected');
                    }
                    if (node.hasAttribute && node.hasAttribute('contenteditable')) {
                        node.setAttribute('contenteditable', 'true');
                    }
                    if (node.querySelectorAll) {
                        node.querySelectorAll('.exam-protected').forEach(el => el.classList.remove('exam-protected'));
                        node.querySelectorAll('[contenteditable="false"]').forEach(el => el.setAttribute('contenteditable', 'true'));
                    }
                }
            });
        });
    });
    domObserver.observe(document.body, { childList: true, subtree: true });

    console.log('Bloqueio desabilitado! Se divirta.');
})();
```
