# Poop
Script 
-- Script AutoTouch: 100% de precisão ao clicar quando a barra branca cruza a vermelha
-- Loop infinito + botão "Parar"

-- === CONFIGURAÇÕES FIXAS ===
local cx, cy = 980, 540               -- Botão "Carregar"
local bx, by = 710, 505               -- Início da barra
local largura = 500                   -- Largura da barra
local corVerde = 0x4ADA58             -- Cor barra carregando
local corBranca = 0xFFFFFF            -- Cor barra móvel
local corVermelha = 0xFF3C3C          -- Cor da área de clique
local tolerancia = 15                 -- Tolerância refinada
local cooldown_ms = 3000              -- Delay entre ciclos

-- === CONTROLE ===
local rodando = true

-- Botão flutuante "Parar"
createHUD(1, "Parar", 50, 100, 150, 60)
showHUD(1, 1)

touchEventCallback(function(event)
    if event.action == 0 then
        if event.x >= 50 and event.x <= 200 and event.y >= 100 and event.y <= 160 then
            rodando = false
            toast("Automação parada.")
            hideHUD(1)
        end
    end
end)

-- Função: converte cor para RGB e compara
function corAproximada(c1, c2, tol)
    local function rgb(c)
        return math.floor(c / 0x10000), math.floor((c / 0x100) % 0x100), c % 0x100
    end
    local r1, g1, b1 = rgb(c1)
    local r2, g2, b2 = rgb(c2)
    return math.abs(r1 - r2) <= tol and math.abs(g1 - g2) <= tol and math.abs(b1 - b2) <= tol
end

-- === LOOP INFINITO ===
while rodando do
    toast("Carregando...")

    -- Esperar barra carregar totalmente (verde)
    while rodando do
        -- Clica no botão "Carregar"
        touchDown(1, cx, cy)
        mSleep(50)
        touchUp(1, cx, cy)
        mSleep(300)

        -- Verifica se a barra está toda verde
        local cheia = true
        for i = 0, largura, 5 do
            local px = bx + i
            local cor = getColor(px, by)
            if not corAproximada(cor, corVerde, tolerancia) then
                cheia = false
                break
            end
        end
        if cheia then break end
    end

    -- Espera o momento certo para clicar na barra branca quando passar pela zona vermelha
    toast("Esperando ponto exato...")
    local clicou = false

    while rodando and not clicou do
        for i = 0, largura, 1 do
            local px = bx + i
            local corAtual = getColor(px, by)

            -- Verifica se está branco (barra móvel) e se está dentro da zona vermelha
            if corAproximada(corAtual, corBranca, tolerancia) and corAproximada(getColor(px, by), corVermelha, tolerancia + 5) then
                -- Clique certeiro
                touchDown(1, cx, cy)
                mSleep(35)
                touchUp(1, cx, cy)
                toast("🎯 Clique preciso!")
                clicou = true
                break
            end
        end
        mSleep(5)
    end

    -- Espera cooldown antes de repetir
    if rodando then
        toast("Aguardando "..(cooldown_ms / 1000).."s...")
        mSleep(cooldown_ms)
    end
end

toast("Script encerrado.")
