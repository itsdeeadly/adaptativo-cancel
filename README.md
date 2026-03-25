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

    // 2. Lógica de extração e cópia
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
            alert('Sucesso!\n\nFormato:\n*TEXTO 1*\n...\n*PERGUNTA*\n...\n*ALTERNATIVA A*\n...');
        } catch (err) {
            console.error(err);
            alert('Erro ao copiar: ' + err.message);
        }
    };

    document.body.appendChild(btn);
    console.log('Verifique o canto inferior direito.');
})();
```
