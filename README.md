# adaptativo-cancel

Copie e cole no console o seguinte comando para desativar o **bloqueio de copia e cola do site** *"Adaptativo SESI"*

```
(function() {
    // 1. Força seleção de texto em todo o documento
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

    // 2. Remove classe .exam-protected e atributos restritivos
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

    // 4. Remove quaisquer listeners globais antigos
    window.oncopy = null;
    document.oncopy = null;
    document.onselectstart = null;
    document.ondragstart = null;

    // 5. Substitui addEventListener para impedir novos bloqueios
    const originalAddEventListener = EventTarget.prototype.addEventListener;
    EventTarget.prototype.addEventListener = function(type, listener, options) {
        const blockedTypes = ['copy', 'cut', 'paste', 'selectstart', 'dragstart'];
        if (blockedTypes.includes(type)) {
            console.log(`[desbloqueio] Listener de ${type} bloqueado.`);
            return;
        }
        return originalAddEventListener.call(this, type, listener, options);
    };

    // 6. Intercepta eventos de cópia em fase de captura e garante que não sejam cancelados
    const allowCopy = (e) => {
        e.stopImmediatePropagation();
        // Não chama preventDefault() – o comportamento padrão da cópia ocorre
    };
    document.addEventListener('copy', allowCopy, true);
    document.addEventListener('cut', allowCopy, true);
    document.addEventListener('paste', allowCopy, true);

    // 7. Remove toasts de notificação que aparecem ao copiar
    const removeToasts = () => {
        document.querySelectorAll('.Toastify__toast').forEach(t => t.remove());
    };
    removeToasts();
    const toastObserver = new MutationObserver(removeToasts);
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

    // 9. (Opcional) Adiciona um botão flutuante para copiar a questão atual
    const addCopyButton = () => {
        // Tenta encontrar o container da questão (ajuste o seletor conforme necessário)
        const questionContainer = document.querySelector('[data-testid="exam-page-root"] .css-vej38e, .canvas-target-content');
        if (!questionContainer) return;

        const button = document.createElement('button');
        button.textContent = 'Copiar questão';
        button.style.position = 'fixed';
        button.style.bottom = '20px';
        button.style.right = '20px';
        button.style.zIndex = '10000';
        button.style.padding = '10px 15px';
        button.style.backgroundColor = '#30AF4A';
        button.style.color = 'white';
        button.style.border = 'none';
        button.style.borderRadius = '5px';
        button.style.cursor = 'pointer';
        button.style.fontSize = '14px';
        button.style.boxShadow = '0 2px 5px rgba(0,0,0,0.2)';

        button.onclick = () => {
            // Seleciona o texto da questão atual
            const questionText = questionContainer.innerText;
            navigator.clipboard.writeText(questionText).then(() => {
                alert('Questão copiada para a área de transferência!');
            }).catch(err => console.error('Erro ao copiar:', err));
        };
        document.body.appendChild(button);
    };

    // Aguarda um pouco para garantir que o container exista
    setTimeout(addCopyButton, 1000);

    console.log('Bloqueio desabilitado! Se divirta.');
    console.log('Criado o botão do caos. Cuidado.');
})();
```
