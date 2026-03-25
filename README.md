# adaptativo-cancel

Copie e cole no console o seguinte comando para desativar o **bloqueio de copia e cola do site** *"Adaptativo SESI"*

```
(function() {
    // Evita criar múltiplos botões
    if (document.getElementById('copy-question-btn')) return;

    // 1. Cria o botão
    const btn = document.createElement('button');
    btn.id = 'copy-question-btn';
    btn.textContent = 'Questão';
    btn.style.cssText = `
        position: fixed;
        bottom: 20px;
        right: 20px;
        z-index: 9999;
        background-color: #30AF4A;
        color: white;
        border: none;
        border-radius: 8px;
        padding: 10px 16px;
        font-size: 14px;
        font-weight: bold;
        cursor: pointer;
        box-shadow: 0 2px 8px rgba(0,0,0,0.2);
        transition: all 0.2s;
    `;
    btn.onmouseover = () => btn.style.backgroundColor = '#258f3c';
    btn.onmouseout = () => btn.style.backgroundColor = '#30AF4A';

    // 2. Função para mostrar o balão de fala "Copiado!"
    function showCopiedMessage(button) {
        // Remove qualquer pop-up existente para evitar duplicação
        const existingPopup = document.querySelector('.copy-success-popup');
        if (existingPopup) existingPopup.remove();

        // Cria o elemento do balão
        const popup = document.createElement('div');
        popup.className = 'copy-success-popup';
        popup.textContent = 'Copiado!';

        // Adiciona o estilo CSS (caso ainda não exista)
        if (!document.getElementById('popup-styles')) {
            const style = document.createElement('style');
            style.id = 'popup-styles';
            style.textContent = `
                .copy-success-popup {
                    position: fixed;
                    background-color: #333;
                    color: #fff;
                    padding: 8px 16px;
                    border-radius: 12px;
                    font-size: 14px;
                    font-weight: bold;
                    white-space: nowrap;
                    z-index: 10000;
                    box-shadow: 0 4px 12px rgba(0,0,0,0.2);
                    pointer-events: none;
                    opacity: 0;
                    transition: opacity 0.2s ease;
                }
                .copy-success-popup::after {
                    content: '';
                    position: absolute;
                    top: 100%;
                    left: 50%;
                    transform: translateX(-50%);
                    border-width: 6px;
                    border-style: solid;
                    border-color: #333 transparent transparent transparent;
                }
            `;
            document.head.appendChild(style);
        }

        document.body.appendChild(popup);

        // Posiciona acima do botão
        const rect = button.getBoundingClientRect();
        const popupHeight = popup.offsetHeight;
        const popupWidth = popup.offsetWidth;
        popup.style.top = `${rect.top - popupHeight - 10}px`;
        popup.style.left = `${rect.left + rect.width / 2 - popupWidth / 2}px`;

        // Força reflow para então mostrar com fade
        popup.offsetHeight;
        popup.style.opacity = '1';

        // Remove após 2 segundos
        setTimeout(() => {
            popup.style.opacity = '0';
            setTimeout(() => popup.remove(), 200);
        }, 2000);
    }

    // 3. Lógica de extração e cópia com pop-up no lugar do alert
    btn.onclick = async () => {
        try {
            // Container principal da questão
            const examContainer = document.querySelector('[data-testid="exam-page-root"]') ||
                                  document.querySelector('.css-vej38e');
            if (!examContainer) {
                alert('Não foi possível localizar a questão.');
                return;
            }

            // Encontra todas as alternativas
            const alternatives = Array.from(examContainer.querySelectorAll('.css-1q6aftr'));
            if (!alternatives.length) {
                alert('Nenhuma alternativa encontrada.');
                return;
            }

            // Encontra todos os blocos de texto .tiptap que estão fora das alternativas
            const allTiptap = Array.from(examContainer.querySelectorAll('.tiptap'));
            const tiptapOutside = allTiptap.filter(el => !alternatives.some(alt => alt.contains(el)));

            // Separa: os anteriores ao último são os textos principais; o último é a pergunta
            const textBlocks = tiptapOutside.slice(0, -1);
            const questionBlock = tiptapOutside[tiptapOutside.length - 1];

            // Constrói o conteúdo formatado
            let output = '';

            // Textos principais
            textBlocks.forEach((block, idx) => {
                const content = block.innerText.trim();
                if (content) {
                    output += `*TEXTO ${idx + 1}*\n${content}\n\n`;
                }
            });

            // Pergunta
            if (questionBlock) {
                const questionText = questionBlock.innerText.trim();
                if (questionText) {
                    output += `*PERGUNTA*\n${questionText}\n\n`;
                }
            }

            // Alternativas
            alternatives.forEach((alt, idx) => {
                const letterSpan = alt.querySelector('.css-1qf2sxb');
                const letter = letterSpan ? letterSpan.innerText.trim() : String.fromCharCode(65 + idx);
                const contentDiv = alt.querySelector('.tiptap');
                const content = contentDiv ? contentDiv.innerText.trim() : '';
                if (content) {
                    output += `*ALTERNATIVA ${letter}*\n${content}\n\n`;
                }
            });

            if (!output.trim()) {
                alert('Nenhum conteúdo extraído.');
                return;
            }

            // Copia para a área de transferência
            await navigator.clipboard.writeText(output);

            // Exibe o balão de "Copiado!" em vez do alert
            showCopiedMessage(btn);
        } catch (err) {
            console.error(err);
            alert('Erro ao copiar: ' + err.message);
        }
    };

    document.body.appendChild(btn);
    console.log('Verifique o canto inferior direito.');
})();
