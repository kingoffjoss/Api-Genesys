// ==UserScript==
// @name          Gerenciador de Conversas com Cronômetro e Numeração
// @namespace     http://seu-dominio.com.br/ (pode ser o seu site, ou deixar como está)
// @version       1.8 // Versão atualizada: Número da conversa alinhado mais à direita
// @description   Adiciona cronômetros e numeração dinâmica a conversas em plataforma web, com formato de tempo simplificado e menor, e símbolo de checkmark estilizado quando concluído. O número da conversa permanece visível e alinhado ao cronômetro.
// @author        Seu Nome (ou Gemini)
// @match         https://apps.sae1.pure.cloud/directory/#/activity* // <<< MUITO IMPORTANTE: SUBSTITUA PELA URL REAL DA SUA PLATAFORMA!
// @grant         none
// ==/UserScript==

(function() {
    // --- Configurações ---
    const limitTimeMs = 70 * 1000;
    const updateIntervalMs = 10;
    const formatTime = num => String(num).padStart(2, '0');

    // Estilos para o cronômetro e número da conversa
    const runningBgColor = 'rgba(255, 255, 255, 0.9)';
    const runningTextColor = 'black';
    const runningBorderColor = '#ccc';
    const runningBoxShadow = '0 1px 3px rgba(0,0,0,0.1)';

    const timerFontSize = '12px';
    const timerPadding = '2px 5px';
    const timerBorderRadius = '3px';

    const pausedBgColor = 'rgba(255, 255, 255, 0.9)';
    const pausedTextColor = 'red';
    const pausedBorderColor = '#ccc';
    const pausedBoxShadow = '0 1px 3px rgba(0,0,0,0.1)';

    const completedBgColor = 'rgba(0, 0, 0, 1)'; // Fundo preto sólido
    const completedTextColor = 'lime'; // Cor verde para o checkmark
    const completedText = '✓';
    const completedFontSize = '20px';
    const completedPadding = '0px';
    const completedBorderRadius = '5px';

    const numberBgColor = runningBgColor;
    const numberTextColor = runningTextColor;
    const numberBorderColor = runningBorderColor;
    const numberBoxShadow = runningBoxShadow;
    const numberFontSize = '12px';
    const numberPadding = '2px 5px';
    const numberBorderRadius = '3px';

    // ALTERAÇÃO AQUI: Ajusta a posição à direita para alinhar com o cronômetro
    const numberRightPosition = '5px'; // Reduzido de 15px para 5px (mesma do cronômetro)
    const numberTopPosition = '2px';

    const conversationTimers = new Map();
    let currentActiveConversationEl = null;

    // --- Funções de Controle de Cronômetro Individual ---
    function renderTime(timerDiv, elapsedTime) {
        let totalSeconds = Math.floor(elapsedTime / 1000);
        let minutes = Math.floor(totalSeconds / 60);
        let seconds = totalSeconds % 60;
        timerDiv.textContent = `${formatTime(minutes)}:${formatTime(seconds)}`;
    }

    function pauseTimer(conversationEl) {
        const timerData = conversationTimers.get(conversationEl);
        if (timerData && !timerData.isPaused) {
            clearInterval(timerData.timerIntervalId);
            timerData.pausedAt = Date.now();
            timerData.isPaused = true;
            if (!timerData.timerDiv.classList.contains('timer-completed')) {
                timerData.timerDiv.style.backgroundColor = pausedBgColor;
                timerData.timerDiv.style.color = pausedTextColor;
                timerData.timerDiv.style.borderColor = pausedBorderColor;
                timerData.timerDiv.style.boxShadow = pausedBoxShadow;
            }
        }
    }

    function resumeTimer(conversationEl) {
        const timerData = conversationTimers.get(conversationEl);
        if (timerData && timerData.isPaused && !timerData.timerDiv.classList.contains('timer-completed')) {
            timerData.startTime += (Date.now() - timerData.pausedAt);
            timerData.isPaused = false;
            timerData.timerIntervalId = setInterval(() => updateIndividualTimer(conversationEl), updateIntervalMs);
            timerData.timerDiv.style.backgroundColor = runningBgColor;
            timerData.timerDiv.style.color = runningTextColor;
            timerData.timerDiv.style.borderColor = runningBorderColor;
            timerData.timerDiv.style.boxShadow = runningBoxShadow;
        }
    }

    function removeTimer(conversationEl) {
        const timerData = conversationTimers.get(conversationEl);
        if (timerData) {
            clearInterval(timerData.timerIntervalId);
            if (timerData.timerDiv && timerData.timerDiv.parentNode) {
                timerData.timerDiv.remove();
            }
            if (timerData.numberDiv && timerData.numberDiv.parentNode) {
                timerData.numberDiv.remove();
            }
            conversationTimers.delete(conversationEl);
        }
    }

    function updateIndividualTimer(conversationEl) {
        const timerData = conversationTimers.get(conversationEl);
        if (!timerData || timerData.isPaused) return;

        let elapsedTime = Date.now() - timerData.startTime;

        if (elapsedTime >= limitTimeMs) {
            clearInterval(timerData.timerIntervalId);
            timerData.timerDiv.textContent = completedText;
            timerData.timerDiv.style.backgroundColor = completedBgColor;
            timerData.timerDiv.style.color = completedTextColor;
            timerData.timerDiv.style.fontSize = completedFontSize;
            timerData.timerDiv.style.borderColor = 'transparent';
            timerData.timerDiv.style.boxShadow = 'none';
            timerData.isPaused = true;
            timerData.timerDiv.classList.add('timer-completed');

            timerData.timerDiv.style.top = '50%';
            timerData.timerDiv.style.right = '5px';
            timerData.timerDiv.style.left = 'auto';
            timerData.timerDiv.style.transform = 'translateY(-50%)';
            timerData.timerDiv.style.width = 'fit-content';
            timerData.timerDiv.style.textAlign = 'center';
            timerData.timerDiv.style.padding = completedPadding;
            timerData.timerDiv.style.borderRadius = completedBorderRadius;

            console.log(`Cronômetro da conversa (ID: ${conversationEl.id || 'N/A'}) concluído: ✓`);
            return;
        }
        renderTime(timerData.timerDiv, elapsedTime);
    }

    function createTimerAndNumberForConversation(conversationEl) {
        let existingTimerDivInDOM = conversationEl.querySelector('.injected-conversation-timer');
        let existingNumberDivInDOM = conversationEl.querySelector('.injected-conversation-number');

        if (existingTimerDivInDOM || existingNumberDivInDOM) {
            if (conversationTimers.has(conversationEl)) {
                const existingTimerData = conversationTimers.get(conversationEl);
                if (existingTimerData.timerDiv.classList.contains('timer-completed')) {
                    clearInterval(existingTimerData.timerIntervalId);
                    existingTimerData.startTime = Date.now();
                    existingTimerData.pausedAt = Date.now();
                    existingTimerData.isPaused = true;
                    renderTime(existingTimerData.timerDiv, 0);
                    existingTimerData.timerDiv.style.backgroundColor = pausedBgColor;
                    existingTimerData.timerDiv.style.color = pausedTextColor;
                    existingTimerData.timerDiv.style.fontSize = timerFontSize;
                    existingTimerData.timerDiv.style.borderColor = pausedBorderColor;
                    existingTimerData.timerDiv.style.boxShadow = pausedBoxShadow;
                    existingTimerData.timerDiv.classList.remove('timer-completed');
                    existingTimerData.timerDiv.style.padding = timerPadding;
                    existingTimerData.timerDiv.style.borderRadius = timerBorderRadius;

                    existingTimerData.timerDiv.style.top = '50%';
                    existingTimerData.timerDiv.style.right = '5px';
                    existingTimerData.timerDiv.style.left = 'auto';
                    existingTimerData.timerDiv.style.transform = 'translateY(-50%)';
                    existingTimerData.timerDiv.style.width = 'fit-content';
                    existingTimerData.timerDiv.style.textAlign = 'center';
                    existingTimerData.timerDiv.style.minWidth = '45px';

                    if (existingTimerData.numberDiv) {
                        existingTimerData.numberDiv.style.display = 'block';
                        existingTimerData.numberDiv.style.top = numberTopPosition;
                        existingTimerData.numberDiv.style.right = numberRightPosition; // Ajustado aqui também
                        existingTimerData.numberDiv.style.left = 'auto';
                        existingTimerData.numberDiv.style.transform = 'none';
                    }
                }
                return existingTimerData;
            } else {
                console.warn("Script: Encontrado timer/número injetado 'órfão' no DOM para uma conversa. Removendo para recriar.");
                if (existingTimerDivInDOM) existingTimerDivInDOM.remove();
                if (existingNumberDivInDOM) existingNumberDivInDOM.remove();
            }
        }
        if (conversationTimers.has(conversationEl)) {
             return conversationTimers.get(conversationEl);
        }

        conversationEl.style.position = 'relative';

        let timerDiv = document.createElement('div');
        timerDiv.className = 'injected-conversation-timer';
        timerDiv.style.position = 'absolute';
        timerDiv.style.top = '50%';
        timerDiv.style.right = '5px';
        timerDiv.style.left = 'auto';
        timerDiv.style.transform = 'translateY(-50%)';
        timerDiv.style.backgroundColor = pausedBgColor;
        timerDiv.style.color = pausedTextColor;
        timerDiv.style.padding = timerPadding;
        timerDiv.style.borderRadius = timerBorderRadius;
        timerDiv.style.fontSize = timerFontSize;
        timerDiv.style.fontFamily = 'monospace';
        timerDiv.style.zIndex = '9999';
        timerDiv.style.width = 'fit-content';
        timerDiv.style.textAlign = 'center';
        timerDiv.style.border = `1px solid ${pausedBorderColor}`;
        timerDiv.style.boxShadow = pausedBoxShadow;
        timerDiv.style.fontWeight = 'bold';
        timerDiv.style.minWidth = '45px';
        conversationEl.appendChild(timerDiv);

        let numberDiv = document.createElement('div');
        numberDiv.className = 'injected-conversation-number';
        numberDiv.style.position = 'absolute';
        numberDiv.style.top = numberTopPosition;
        numberDiv.style.right = numberRightPosition; // AQUI ESTÁ A ALTERAÇÃO PRINCIPAL
        numberDiv.style.left = 'auto';
        numberDiv.style.transform = 'none';
        numberDiv.style.backgroundColor = numberBgColor;
        numberDiv.style.color = numberTextColor;
        numberDiv.style.padding = numberPadding;
        numberDiv.style.borderRadius = numberBorderRadius;
        numberDiv.style.fontSize = numberFontSize;
        numberDiv.style.fontFamily = 'sans-serif';
        numberDiv.style.fontWeight = 'bold';
        numberDiv.style.zIndex = '99999';
        numberDiv.style.width = 'fit-content';
        numberDiv.style.textAlign = 'center';
        numberDiv.style.minWidth = '20px';
        numberDiv.style.border = `1px solid ${numberBorderColor}`;
        numberDiv.style.boxShadow = numberBoxShadow;
        numberDiv.style.display = 'block';
        conversationEl.appendChild(numberDiv);

        const startTime = Date.now();
        const timerData = {
            timerDiv: timerDiv,
            numberDiv: numberDiv,
            startTime: startTime,
            pausedAt: Date.now(),
            timerIntervalId: null,
            isPaused: true
        };
        conversationTimers.set(conversationEl, timerData);

        renderTime(timerDiv, 0);
        return timerData;
    }

    function updateConversationNumbers() {
        const existingConversations = Array.from(document.querySelectorAll('div.interaction-group'))
            .filter(el => conversationTimers.has(el));

        existingConversations.sort((a, b) => b.offsetTop - a.offsetTop);

        existingConversations.forEach((conversationEl, index) => {
            const timerData = conversationTimers.get(conversationEl);
            if (timerData && timerData.numberDiv) {
                // O número deve sempre ser exibido, independentemente do status do timer
                timerData.numberDiv.textContent = (index + 1).toString();
                timerData.numberDiv.style.backgroundColor = numberBgColor;
                timerData.numberDiv.style.color = numberTextColor;
                timerData.numberDiv.style.fontSize = numberFontSize;
                timerData.numberDiv.style.display = 'block'; // Garante que seja sempre 'block'
                timerData.numberDiv.style.border = `1px solid ${numberBorderColor}`;
                timerData.numberDiv.style.boxShadow = numberBoxShadow;
                timerData.numberDiv.style.right = numberRightPosition; // Garante que a posição seja atualizada também
            }
        });

        // Limpa timers de conversas que não estão mais no DOM
        conversationTimers.forEach((timerData, conversationEl) => {
            if (!document.body.contains(conversationEl)) {
                if (timerData.numberDiv) {
                    timerData.numberDiv.remove();
                }
                clearInterval(timerData.timerIntervalId);
                conversationTimers.delete(conversationEl);
            }
        });
    }

    function manageActiveTimers() {
        const currentlySelectedConversation = document.querySelector('div.interaction-group.is-selected');

        conversationTimers.forEach((timerData, conversationEl) => {
            if (conversationEl !== currentlySelectedConversation && !timerData.isPaused && !timerData.timerDiv.classList.contains('timer-completed')) {
                pauseTimer(conversationEl);
            }
        });

        if (currentlySelectedConversation) {
            if (!conversationTimers.has(currentlySelectedConversation)) {
                createTimerAndNumberForConversation(currentlySelectedConversation);
            }
            if (!conversationTimers.get(currentlySelectedConversation).timerDiv.classList.contains('timer-completed')) {
                 resumeTimer(currentlySelectedConversation);
            }
            currentActiveConversationEl = currentlySelectedConversation;
        } else {
            currentActiveConversationEl = null;
        }
    }

    // --- Lógica de Inicialização e Eventos ---
    // Remove quaisquer elementos injetados de uma execução anterior (útil para recarga do script)
    document.querySelectorAll('.injected-conversation-timer, .injected-conversation-number').forEach(el => el.remove());
    conversationTimers.clear();

    // Pequeno atraso para garantir que os elementos iniciais da página estejam carregados
    setTimeout(() => {
        const initialConversationElements = document.querySelectorAll('div.interaction-group');
        if (initialConversationElements.length === 0) {
            console.warn("Nenhum elemento de conversa com a classe 'interaction-group' encontrado na carga inicial. Isso pode ser normal se as conversas carregarem dinamicamente. O script continuará monitorando.");
        } else {
            initialConversationElements.forEach(createTimerAndNumberForConversation);
            console.log(`Encontrados ${initialConversationElements.length} conversas existentes ao iniciar.`);
        }
        updateConversationNumbers(); // Atualiza os números logo no início
        manageActiveTimers();       // Gerencia o estado dos timers
    }, 500); // Meio segundo de atraso

    const conversationsListContainer = document.body; // Observa o body para detectar mudanças em toda a estrutura
    const observer = new MutationObserver((mutationsList, observer) => {
        let shouldUpdateNumbers = false;
        let shouldManageTimers = false;

        for (const mutation of mutationsList) {
            if (mutation.type === 'childList') {
                if (mutation.addedNodes.length > 0) {
                    for (const node of mutation.addedNodes) {
                        // Verifica se o nó adicionado é uma conversa ou contém conversas
                        if (node.nodeType === 1) { // Garante que é um Element
                            if (node.matches('div.interaction-group')) {
                                createTimerAndNumberForConversation(node);
                                shouldUpdateNumbers = true;
                                shouldManageTimers = true;
                            }
                            // Busca por conversas dentro dos nós adicionados (ex: um grande bloco de HTML foi adicionado)
                            const newConversationElements = node.querySelectorAll('div.interaction-group');
                            newConversationElements.forEach(newConvEl => {
                                if (!conversationTimers.has(newConvEl)) {
                                    createTimerAndNumberForConversation(newConvEl);
                                    shouldUpdateNumbers = true;
                                    shouldManageTimers = true;
                                }
                            });
                        }
                    }
                }
                if (mutation.removedNodes.length > 0) {
                    for (const node of mutation.removedNodes) {
                        // Remove timers de conversas que foram removidas do DOM
                        if (node.nodeType === 1 && node.matches('div.interaction-group')) {
                            removeTimer(node);
                            shouldUpdateNumbers = true;
                            shouldManageTimers = true;
                        }
                    }
                }
            }

            if (mutation.type === 'attributes' && mutation.attributeName === 'class') {
                const changedElement = mutation.target;
                // Se a classe de uma conversa mudou (ex: para 'is-selected')
                if (changedElement.matches('div.interaction-group')) {
                    shouldManageTimers = true;
                }
            }
        }
        // Executa as atualizações necessárias apenas uma vez por ciclo de mutação
        if (shouldUpdateNumbers) {
            updateConversationNumbers();
        }
        if (shouldManageTimers) {
            manageActiveTimers();
        }
    });

    // Inicia a observação de mudanças no DOM
    observer.observe(conversationsListContainer, { childList: true, subtree: true, attributes: true, attributeFilter: ['class'] });

    console.log("Script de gerenciamento de cronômetros e numeração de conversas injetado. Ativação via classe 'is-selected'.");
})();

// ==UserScript==
// @name         Copiar CPF e Interação
// @namespace    http://tampermonkey.net/
// @version      2.1
// @description  Copia o CPF ou a interação selecionada para a área de transferência (versão otimizada)
// @author       Lucas Vinicius
// @match        https://apps.sae1.pure.cloud/*
// @run-at       document-end
// @grant        GM_addStyle
// ==/UserScript==

(function() {
    'use strict';

    // Cache de elementos DOM frequentemente acessados
    const domCache = new WeakMap();
    const getCache = {
        toolbarDiv: null,
        buttonCPF: null,
        buttonInteraction: null
    };

    // Constantes para validação de documentos
    const INVALID_DOCUMENTS = new Set([
        "08002813017", "08002814000", "61271781919", "41271781919",
        "91271781919", "81271781919", "71271781919", "51271781919",
        "31271781919", "21271781919", "11271781919", "01271781919",
        "00000000000", "11111111111", "22222222222", "33333333333",
        "44444444444", "55555555555", "66666666666", "77777777777",
        "88888888888", "99999999999"
    ]);

    // Funções de validação otimizadas
    const documentValidator = {
        isRepeatedDigits: str => /^(\d)\1+$/.test(str),

        validateCPF: cpf => {
            cpf = cpf.replace(/\D/g, '');
            if (cpf.length !== 11) return false;
            if (INVALID_DOCUMENTS.has(cpf)) return false;
            if (documentValidator.isRepeatedDigits(cpf)) return false;

            const calcDigit = (slice, weights) => {
                const sum = slice.split('').reduce((acc, digit, idx) =>
                    acc + parseInt(digit) * weights[idx], 0);
                const rest = (sum * 10) % 11;
                return (rest === 10 || rest === 11) ? 0 : rest;
            };

            const weights1 = [10, 9, 8, 7, 6, 5, 4, 3, 2];
            const weights2 = [11, 10, 9, 8, 7, 6, 5, 4, 3, 2];

            const digit1 = calcDigit(cpf.slice(0, 9), weights1);
            if (digit1 !== parseInt(cpf[9])) return false;

            const digit2 = calcDigit(cpf.slice(0, 10), weights2);
            return digit2 === parseInt(cpf[10]);
        },

        extractPossibleCPFs: text => {
            const sanitizedText = text.replace(/[^\d.-]/g, ' ');
            const patterns = [
                /\d{3}\.?\d{3}\.?\d{3}-?\d{2}/g,
                /\b\d{11}\b/g,
            ];

            const possibleCPFs = new Set();

            patterns.forEach(pattern => {
                const matches = sanitizedText.match(pattern);
                if (matches) {
                    matches.forEach(match => {
                        const cpf = match.replace(/\D/g, '');
                        if (documentValidator.validateCPF(cpf)) {
                            possibleCPFs.add(cpf);
                        }
                    });
                }
            });

            return Array.from(possibleCPFs);
        }
    };

    // Função melhorada para processar o painel de scripts
    async function processScriptPanel() {
        try {
            const toggleButton = document.querySelector("button.toggleItem-scripts");
            if (toggleButton &&
                !toggleButton.classList.contains("active") &&
                toggleButton.getAttribute("aria-expanded") !== "true") {
                toggleButton.click();
                await new Promise(resolve => setTimeout(resolve, 250));
            }

            const scriptIframe = document.querySelector(".interaction-script-container.no-headers.non-call iframe");
            if (scriptIframe?.contentDocument) {
                const spans = scriptIframe.contentDocument.querySelectorAll("span.text-component");
                const documentosEncontrados = new Set();

                spans.forEach(span => {
                    const cpfs = documentValidator.extractPossibleCPFs(span.textContent);
                    cpfs.forEach(cpf => documentosEncontrados.add(cpf));
                });

                return Array.from(documentosEncontrados);
            }

            return [];
        } catch (error) {
            console.error('Erro ao processar painel de scripts:', error);
            return [];
        }
    }

    // Função otimizada para encontrar iframes de mensagem ativos
    async function findActiveMessagingIframes() {
        const appViewStack = document.querySelector('app-view-stack[name="conversation"]');
        if (!appViewStack?.shadowRoot) return [];

        const iframes = Array.from(appViewStack.shadowRoot.querySelectorAll('iframe[src*="messaging-gadget"]'));

        return iframes.filter(iframe => {
            const instance = iframe.closest('.app-instance');
            const isVisible = instance && instance.getAttribute('aria-hidden') !== 'true';
            const hasContent = iframe?.contentDocument?.body?.textContent?.trim().length > 0;
            return isVisible && hasContent;
        });
    }

    // Função otimizada para extrair documentos do texto
    function extractDocumentoFromText(text) {
        const sanitizedText = text.replace(/\D/g, "");

        if (sanitizedText.length === 11 && documentValidator.validateCPF(sanitizedText)) {
            return sanitizedText;
        }

        if (sanitizedText.length === 14 && documentValidator.validateCNPJ(sanitizedText)) {
            return sanitizedText;
        }

        return null;
    }

    // Função assíncrona para encontrar documentos com melhor performance
    async function encontrarDocumentos() {
        const documentosEncontrados = new Set();

        try {
            // Primeiro, procura no painel de scripts
            const scriptDocuments = await processScriptPanel();
            scriptDocuments.forEach(doc => documentosEncontrados.add(doc));

            // Se não encontrou nada no painel de scripts, procura nos iframes de mensagem
            if (documentosEncontrados.size === 0) {
                const activeIframes = await findActiveMessagingIframes();
                for (const iframe of activeIframes) {
                    if (iframe?.contentDocument) {
                        const messages = iframe.contentDocument.querySelectorAll(
                            '._text_lfiei_1[data-automation-id="messaging-gadget-message-textbody"]'
                        );

                        messages.forEach(message => {
                            const cpfs = documentValidator.extractPossibleCPFs(message.textContent);
                            cpfs.forEach(cpf => documentosEncontrados.add(cpf));
                        });

                        const nameElements = iframe.contentDocument.querySelectorAll(
                            '._name_1bevi_42, .gux-truncate'
                        );

                        nameElements.forEach(element => {
                            const cpfs = documentValidator.extractPossibleCPFs(element.textContent);
                            cpfs.forEach(cpf => documentosEncontrados.add(cpf));
                        });
                    }
                }
            }

            return Array.from(documentosEncontrados);
        } catch (error) {
            console.error('Erro ao procurar documentos:', error);
            return [];
        }
    }

    // UI Components otimizados
        const UI = {
        createNotification: (message, type = 'info') => {
            const notification = document.createElement("div");
            notification.textContent = message;
            notification.style.cssText = `
                position: fixed;
                top: 50%;
                left: 50%;
                transform: translate(-50%, -50%);
                background-color: ${type === 'error' ? '#ff4444' : '#333'};
                color: #fff;
                padding: 10px 20px;
                border-radius: 5px;
                z-index: 9999;
                font-size: 14px;
                box-shadow: 0 2px 5px rgba(0,0,0,0.2);
            `;
            document.body.appendChild(notification);
            setTimeout(() => notification.remove(), 3000);
        },

        createDocumentPopup: (documentos) => {
            const popup = document.createElement("div");
            popup.style.cssText = `
                position: fixed;
    top: 50%;
    left: 50%;
    transform: translate(-50%, -50%);
    background-color: rgb(255, 255, 255);
    color: rgb(0, 0, 0);
    border-radius: 4px;
    z-index: 10000;
    max-height: 300px;
    overflow-y: auto;
    box-shadow: rgba(0, 0, 0, 0.15) 0px 4px 12px;
            `;

            const title = document.createElement("h3");
            title.textContent = "CPFs encontrados";
            title.style.cssText = `
                    font-size: 14px;
    line-height: 18px;
    margin: 0;
    border-bottom: 1px solid #ccc;
    padding: 12px 12px 12px 12px;
    text-align: center;
    cursor: pointer;
            `;
            popup.appendChild(title);

            documentos.forEach(doc => {
                const docItem = document.createElement("div");
                docItem.textContent = doc;
                docItem.style.cssText = `
                    padding: 8px;
                    margin: 4px 0;
                    cursor: pointer;
                    border-radius: 4px;
                    &:hover {
                        background-color: #f5f5f5;
                    }
                `;
                docItem.onclick = async () => {
                    try {
                        await navigator.clipboard.writeText(doc);
                        UI.createNotification("CPF copiado com sucesso!");
                        popup.remove();
                    } catch (error) {
                        UI.createNotification("Erro ao copiar CPF", 'error');
                    }
                };
                popup.appendChild(docItem);
            });

            document.body.appendChild(popup);

            // Fechar popup ao clicar fora dele
            const closePopup = (e) => {
                if (!popup.contains(e.target)) {
                    popup.remove();
                    document.removeEventListener('click', closePopup);
                }
            };
            setTimeout(() => document.addEventListener('click', closePopup), 100);
        }
    };

    // Função otimizada para copiar interação
    async function copyInteraction() {
        const selectedElement = document.querySelector(".is-selected");
        const participantName = selectedElement?.querySelector(".participant-name span")?.textContent.trim();
        const externalParticipantName = document.querySelector("p.participant-name")?.textContent.trim();

        if (participantName !== externalParticipantName) {
            UI.createNotification("Os nomes dos participantes não correspondem.");
            return;
        }

        const copyButton = document.querySelector(".copy-interaction-url-btn");
        if (!copyButton) {
            UI.createNotification("Botão de cópia de interação não encontrado.");
            return;
        }

        copyButton.click();
        await new Promise(resolve => setTimeout(resolve, 150));

        try {
            const url = await navigator.clipboard.readText();
            await navigator.clipboard.writeText(`${participantName} - ${url}`);
            UI.createNotification("Interação copiada com sucesso.");
        } catch (error) {
            UI.createNotification("Erro ao copiar a interação.");
        }
    }

    // Gerenciamento otimizado de botões
    const buttonManager = {
        createButtons() {
            // Verifica se a barra de ferramentas já foi processada
            const toolbarDiv = document.querySelector(
                'div[role="toolbar"][label="Primário"].interaction-controls-primary.toggle-item-container'
            );

            if (!toolbarDiv || domCache.has(toolbarDiv)) return;

            // Criação do botão CPF
            const buttonCPF = document.createElement("button");
            buttonCPF.id = "buttonCPF";
            buttonCPF.innerHTML = `
                <svg xmlns="http://www.w3.org/2000/svg" height="24px" viewBox="0 -960 960 960" width="24px" fill="#5f6368">
                    <path d="M360-240q-33 0-56.5-23.5T280-320v-480q0-33 23.5-56.5T360-880h360q33 0 56.5 23.5T800-800v480q0 33-23.5 56.5T720-240H360Zm0-80h360v-480H360v480ZM200-80q-33 0-56.5-23.5T120-160v-560h80v560h440v80H200Zm160-240v-480 480Z"/>
                </svg>
            `;
            buttonCPF.style.cssText = `
                border: 0;
                height: 40px;
                padding: 0;
                width: 43px;
                background: transparent;
                cursor: pointer;
            `;
            buttonCPF.onclick = async () => {
                const documentos = await encontrarDocumentos();
                if (documentos.length === 0) {
                    UI.createNotification("CPF não encontrado.");
                } else if (documentos.length === 1) {
                    try {
                        await navigator.clipboard.writeText(documentos[0]);
                        UI.createNotification("CPF copiado com sucesso.");
                    } catch {
                        UI.createNotification("Erro ao copiar CPF.");
                    }
                } else {
                    UI.createDocumentPopup(documentos);
                }
            };

            // Criação do botão Interação
            const buttonInteraction = document.createElement("button");
            buttonInteraction.id = "buttonInteraction";
            buttonInteraction.innerHTML = `
                <svg xmlns="http://www.w3.org/2000/svg" height="24px" viewBox="0 -960 960 960" width="24px" fill="#5f6368">
                    <path d="M160-160q-33 0-56.5-23.5T80-240v-480q0-33 23.5-56.5T160-800h200v80H160v480h640v-480H600v-80h200q33 0 56.5 23.5T880-720v480q0 33-23.5 56.5T800-160H160Zm320-184L280-544l56-56 104 104v-304h80v304l104-104 56 56-200 200Z"/>
                </svg>
            `;
            buttonInteraction.style.cssText = `
                border: 0;
                height: 40px;
                padding: 0;
                width: 43px;
                background: transparent;
                cursor: pointer;
            `;
            buttonInteraction.onclick = copyInteraction;

            toolbarDiv.appendChild(buttonCPF);
            toolbarDiv.appendChild(buttonInteraction);

            // Armazenar referências no WeakMap
            domCache.set(toolbarDiv, { buttonCPF, buttonInteraction });
        },

        removeButtons(toolbarDiv) {
            const cachedButtons = domCache.get(toolbarDiv);
            if (cachedButtons) {
                cachedButtons.buttonCPF.remove();
                cachedButtons.buttonInteraction.remove();
                domCache.delete(toolbarDiv);
            }
        }
    };

    // Função de throttle para limitar a frequência de execução
const throttle = (func, limit) => {
    let inThrottle = false;
    return function (...args) {
        if (!inThrottle) {
            func.apply(this, args);
            inThrottle = true;
            setTimeout(() => {
                inThrottle = false;
            }, limit);
        }
    };
};


// Cache de seletores DOM frequentemente utilizados
const SELECTORS = {
    TOOLBAR: 'div[role="toolbar"][label="Primário"].interaction-controls-primary.toggle-item-container',
    END_BUTTON: '.interaction-end-btn'
};

// Função para verificar se uma mutação é relevante
const isRelevantMutation = (mutation) => {
    // Verifica se a mutação afeta elementos que nos interessam
    return Array.from(mutation.addedNodes).some(node => {
        if (!(node instanceof HTMLElement)) return false;

        return (
            // Verifica se o elemento adicionado é a toolbar ou contém a toolbar
            node.matches(SELECTORS.TOOLBAR) ||
            node.querySelector(SELECTORS.TOOLBAR) ||
            // Verifica se é o botão de fim ou contém o botão de fim
            node.matches(SELECTORS.END_BUTTON) ||
            node.querySelector(SELECTORS.END_BUTTON)
        );
    });
};

// Função principal que processa as mutações
    const processMutations = (mutations) => {
        const toolbarDiv = document.querySelector(SELECTORS.TOOLBAR);

        // Cria botões se a toolbar ainda não foi processada
        if (toolbarDiv && !domCache.has(toolbarDiv)) {
            buttonManager.createButtons();
        }
    };


// Aplicando throttle à função de processamento
const throttledProcessMutations = throttle(processMutations, 100);

// Configuração otimizada do observer
const observerConfig = {
    childList: true,
    subtree: true,
    // Não observamos attributes ou characterData, pois não são relevantes
    attributes: false,
    characterData: false
};

// Criação do observer otimizado
const observer = new MutationObserver((mutations) => {
    throttledProcessMutations(mutations);
});

// Iniciando a observação apenas no elemento necessário
observer.observe(document.body, observerConfig);

// Função de limpeza para quando não precisarmos mais do observer
const cleanup = () => {
    observer.disconnect();
};
})();

