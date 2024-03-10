-- Load libraries
local Fluent = loadstring(game:HttpGet("https://raw.githubusercontent.com/title33/zdq/main/README.md"))()
local SaveManager = loadstring(game:HttpGet("https://raw.githubusercontent.com/title33/SaveManager/main/README.md"))()
local InterfaceManager = loadstring(game:HttpGet("https://raw.githubusercontent.com/dawid-scripts/Fluent/master/Addons/InterfaceManager.lua"))()

-- Create a Fluent window
local Window = Fluent:CreateWindow({
    Title = "Xylo Hub",
    SubTitle = "by Sky",
    TabWidth = 160,
    Size = UDim2.fromOffset(580, 460),
    Acrylic = false,
    Theme = "Red",
    MinimizeKey = Enum.KeyCode.LeftControl
})

-- Create tabs and options
local Tabs = {
    General = Window:AddTab({ Title = "General", Icon = "http://www.roblox.com/asset/?id=11254763826" }),
    Settings = Window:AddTab({ Title = "Settings", Icon = "settings" })
}

local Options = Fluent.Options

function TP(cframe)
    GetCurrentVehicle():SetPrimaryPartCFrame(cframe)
end

--Vars
LocalPlayer = game:GetService("Players").LocalPlayer
Camera = workspace.CurrentCamera
VirtualUser = game:GetService("VirtualUser")
MarketplaceService = game:GetService("MarketplaceService")

--Get Current Vehicle
function GetCurrentVehicle()
    return LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("Humanoid") and LocalPlayer.Character.Humanoid.SeatPart and LocalPlayer.Character.Humanoid.SeatPart.Parent
end

--Notification Handler
function SendNotification(Title, Message, Duration)
    game.StarterGui:SetCore("SendNotification", {
        Title = Title;
        Text = Message;
        Duration = Duration;
    })
end

--Regular TP
function TP(cframe)
    GetCurrentVehicle():SetPrimaryPartCFrame(cframe)
end

--Velocity TP
function VelocityTP(cframe)
    TeleportSpeed = math.random(200, 600)
    Car = GetCurrentVehicle()
    local BodyGyro = Instance.new("BodyGyro", Car.PrimaryPart)
    BodyGyro.P = 5000
    BodyGyro.maxTorque = Vector3.new(9e9, 9e9, 9e9)
    BodyGyro.CFrame = Car.PrimaryPart.CFrame
    local BodyVelocity = Instance.new("BodyVelocity", Car.PrimaryPart)
    BodyVelocity.MaxForce = Vector3.new(9e9, 9e9, 9e9)
    BodyVelocity.Velocity = CFrame.new(Car.PrimaryPart.Position, cframe.p).LookVector * TeleportSpeed
    wait((Car.PrimaryPart.Position - cframe.p).Magnitude / TeleportSpeed)
    BodyVelocity.Velocity = Vector3.new()
    wait(0.1)
    BodyVelocity:Destroy()
    BodyGyro:Destroy()
end

--Auto Farm
StartPosition = CFrame.new(Vector3.new(811.013184, 27.3421249, 2203.55542), Vector3.new(-187, 25.7, 1982))
EndPosition = CFrame.new(Vector3.new(-76.4760208, 27.7194824, 1993.84229), Vector3.new(-187, 25.7, 1982))
AutoFarmFunc = coroutine.create(function()
    while wait() do
        if not AutoFarm then
            AutoFarmRunning = false
            coroutine.yield()
        end
        AutoFarmRunning = true
        pcall(function()
            if not GetCurrentVehicle() and tick() - (LastNotif or 0) > 5 then
                LastNotif = tick()
                SendNotification("Aloha Scripts", "Please Enter A Vehicle!")
            else
                TP(StartPosition + (TouchTheRoad and Vector3.new() or Vector3.new(0, 1, 0)))
                VelocityTP(EndPosition + (TouchTheRoad and Vector3.new() or Vector3.new(0, 1, 0)))
                TP(EndPosition + (TouchTheRoad and Vector3.new() or Vector3.new(0, 1, 0)))
                VelocityTP(StartPosition + (TouchTheRoad and Vector3.new() or Vector3.new(0, 1, 0)))
            end
        end)
    end
end)
-- Anti AFK
AntiAFK = true
LocalPlayer.Idled:Connect(function()
    VirtualUser:CaptureController()
    VirtualUser:ClickButton2(Vector2.new(), Camera.CFrame)
end)

-- Add UI elements
local ToggleAutoFarm = Tabs.General:AddToggle("Toggle Auto Farm", {Title = "Auto Farm", Default = false })

ToggleAutoFarm:OnChanged(function(value)
    AutoFarm = value
    if value and not AutoFarmRunning then
        coroutine.resume(AutoFarmFunc)
    end
end)

local InputVelocity = Tabs.General:AddInput("Input Velocity", {
    Title = "Velocity",
    Default = "600",
    Placeholder = "Enter Velocity",
    Numeric = true,
    Finished = false,
    Callback = function(Value)
        print("Velocity changed:", Value)
        SetVelocity(Value)
    end
})

InputVelocity:OnChanged(function()
    print("Velocity updated:", InputVelocity.Value)
end)




-- Load libraries and set up managers
SaveManager:SetLibrary(Fluent)
InterfaceManager:SetLibrary(Fluent)
SaveManager:IgnoreThemeSettings()
SaveManager:SetIgnoreIndexes({})

-- Set folders for SaveManager
InterfaceManager:SetFolder("FluentScriptHub")
SaveManager:SetFolder("FluentScriptHub/specific-game")

-- Build UI sections
InterfaceManager:BuildInterfaceSection(Tabs.Settings)
SaveManager:BuildConfigSection(Tabs.Settings)

-- Select the first tab
Window:SelectTab(1)

-- Load autoload config
SaveManager:LoadAutoloadConfig()
