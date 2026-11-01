local P = game:GetService("Players")
local TC = game:GetService("TextChatService")
local WS = game:GetService("Workspace")
local CoreGui = game:GetService("CoreGui")

local old = CoreGui:FindFirstChild("LinoriaUI")
if old then old:Destroy() end

local repo = 'https://raw.githubusercontent.com/wally-rblx/LinoriaLib/main/'
local Library = loadstring(game:HttpGet(repo .. 'Library.lua'))()
local SaveManager = loadstring(game:HttpGet(repo .. 'addons/SaveManager.lua'))()

local Window = Library:CreateWindow({
    Title = 'SEVEN HUB',
    Center = true,
    AutoShow = true,
})

local Theme = {
    Accent = Color3.fromRGB(255, 200, 0),
    Base = Color3.fromRGB(15, 15, 15),
    Text = Color3.fromRGB(255, 255, 255),
    Outline = Color3.fromRGB(255, 200, 0),
}
Library:SetTheme('Dark')
Library:UpdateTheme(Theme)

local Tabs = {
    Home = Window:AddTab('Home'),
    Code = Window:AddTab('Code'),
    Riddle = Window:AddTab('Riddle'),
    Config = Window:AddTab('Config'),
}

local HomeGroup = Tabs.Home:AddLeftGroupbox('Status')
HomeGroup:AddLabel('Nick: ' .. P.LocalPlayer.Name)
HomeGroup:AddLabel('Modo: Otimizado Mobile')

local CodeGroup = Tabs.Code:AddLeftGroupbox('Auto Code')

CodeGroup:AddToggle('Listen', {
    Text = 'Ouvir Chat/Mapa',
    Default = false,
    Callback = function(v) getgenv().SevenListen = v end
})

CodeGroup:AddToggle('AutoSubmit', {
    Text = 'Auto Submit',
    Default = true,
    Callback = function(v) getgenv().SevenAutoSubmit = v end
})

local CodeInput = CodeGroup:AddTextbox('CodeInput', {
    Text = '',
    Default = 'Aguardando...',
    ClearTextOnFocus = false,
})
getgenv().SevenCodeBox = CodeInput

SaveManager:SetLibrary(Library)
SaveManager:SetFolder('SevenHub')
SaveManager:BuildConfigSection(Tabs['Config'])

local plr = P.LocalPlayer
local jaLidos = {}
local enviando = false
local ultimoCodigo = ""
local ativo = true

local function injetarCodigo(c)
    if enviando or not getgenv().SevenListen then return end
    
    local cod = string.gsub(c, "[%p%c]", ""):upper()
    if cod == ultimoCodigo then return end
    ultimoCodigo = cod
    enviando = true
    
    if getgenv().SevenCodeBox then getgenv().SevenCodeBox:Set(cod) end
    
    local ui = plr.PlayerGui:FindFirstChild("Shop")
    if ui then ui = ui:FindFirstChild("Shop") end
    if ui then ui = ui:FindFirstChild("Content") end
    if ui then ui = ui:FindFirstChild("List") end
    if ui then ui = ui:FindFirstChild("Codes") end
    if ui then ui = ui:FindFirstChild("CodeRedeem") end
    if ui then ui = ui:FindFirstChild("TextBox") end
    
    if ui then
        ui.Text = cod
        task.wait(0.3)
        pcall(function() ui:ReleaseFocus(true) end)
    end
    
    task.wait(3)
    enviando = false
end

TC.MessageReceived:Connect(function(msg)
    if not getgenv().SevenListen or enviando then return end
    local id = msg.MessageId or msg.Text
    if jaLidos[id] then return end
    jaLidos[id] = true
    
    local match = string.match(string.upper(msg.Text), "CODE IS%s+(.+)$") or string.match(string.upper(msg.Text), "CODE:%s*(.+)$")
    if match then injetarCodigo(match) end
end)

task.spawn(function()
    while ativo do
        task.wait(2)
        if not getgenv().SevenListen or enviando then continue end
        
        for _, obj in pairs(WS:GetChildren()) do
            if string.find(string.lower(obj.Name), "sammy") then
                for _, desc in pairs(obj:GetDescendants()) do
                    if desc:IsA("TextLabel") then
                        local match = string.match(string.upper(desc.Text), "CODE IS%s+(.+)") or string.match(string.upper(desc.Text), "CODE:%s*(.+)")
                        if match and not jaLidos[match] then
                            jaLidos[match] = true
                            injetarCodigo(match)
                        end
                    end
                end
            end
        end
    end
end)

Library.Window:GetPropertyChangedSignal("Visible"):Connect(function()
    if not Library.Window.Visible then
        ativo = false
    end
end)
