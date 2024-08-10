_G.HeadSize = 3.5
_G.Disabled = true

local UserInputService = game:GetService("UserInputService")
local RunService = game:GetService("RunService")
local Players = game:GetService("Players")
local LocalPlayer = Players.LocalPlayer

-- Create GUI
local ScreenGui = Instance.new("ScreenGui")
ScreenGui.Parent = game.CoreGui

local Frame = Instance.new("Frame")
Frame.Size = UDim2.new(0, 200, 0, 150)
Frame.Position = UDim2.new(0, 10, 0, 10)
Frame.BackgroundColor3 = Color3.new(0.1, 0.1, 0.1)
Frame.Visible = false
Frame.Parent = ScreenGui

local ToggleESPButton = Instance.new("TextButton")
ToggleESPButton.Size = UDim2.new(0, 180, 0, 50)
ToggleESPButton.Position = UDim2.new(0, 10, 0, 10)
ToggleESPButton.Text = "Toggle head"
ToggleESPButton.Parent = Frame

local HeadSizeLabel = Instance.new("TextLabel")
HeadSizeLabel.Size = UDim2.new(0, 180, 0, 30)
HeadSizeLabel.Position = UDim2.new(0, 10, 0, 70)
HeadSizeLabel.Text = "Head Size: " .. _G.HeadSize
HeadSizeLabel.Parent = Frame

local HeadSizeSlider = Instance.new("TextButton")
HeadSizeSlider.Size = UDim2.new(0, 180, 0, 30)
HeadSizeSlider.Position = UDim2.new(0, 10, 0, 110)
HeadSizeSlider.Text = "Adjust Head Size"
HeadSizeSlider.Parent = Frame

-- Toggle GUI visibility with 'K' key
UserInputService.InputBegan:Connect(function(input, gameProcessed)
    if not gameProcessed and input.KeyCode == Enum.KeyCode.K then
        Frame.Visible = not Frame.Visible
    end
end)

-- Toggle ESP
ToggleESPButton.MouseButton1Click:Connect(function()
    _G.Disabled = not _G.Disabled
end)

-- Adjust Head Size
HeadSizeSlider.MouseButton1Click:Connect(function()
    if _G.HeadSize < 25 then
        _G.HeadSize = _G.HeadSize + 0.5
    else
        _G.HeadSize = 0.5
    end
    HeadSizeLabel.Text = "Head Size: " .. _G.HeadSize
end)

-- ESP Functionality
RunService.RenderStepped:Connect(function()
    if not _G.Disabled then
        for _, v in next, Players:GetPlayers() do
            if v ~= LocalPlayer then  -- Ensure ESP is not applied to the local player
                pcall(function()
                    if v.Character and v.Character:FindFirstChild("Head") then
                        v.Character.Head.Size = Vector3.new(_G.HeadSize, _G.HeadSize, _G.HeadSize)
                        v.Character.Head.Transparency = 0.4  -- Set to 0.3 for semi-transparency
                        v.Character.Head.BrickColor = BrickColor.new("Red")
                        v.Character.Head.Material = "Neon"
                        v.Character.Head.CanCollide = false
                        v.Character.Head.Massless = true
                    end
                end)
            end
        end
    end
end)

local BoxESP = {}

function BoxESP.Create(Player)
    if Player == LocalPlayer then return end  -- Skip creating ESP box for the local player
    if BoxESP[Player] then return end

    local character = Player.Character
    if character and character:IsDescendantOf(workspace) and character:FindFirstChild("Humanoid") and character.Humanoid.Health > 0 then
        local Box = Drawing.new("Square")
        Box.Visible = false
        Box.Color = Color3.fromRGB(255, 255, 255)
        Box.Filled = false
        Box.Thickness = 1

        local Connection
        Connection = RunService.RenderStepped:Connect(function()
            if character and character:IsDescendantOf(workspace) and character:FindFirstChild("Humanoid") and character.Humanoid.Health > 0 then
                local Pos, Visible = workspace.CurrentCamera:WorldToViewportPoint(character.HumanoidRootPart.Position)
                local scale_factor = 1 / (Pos.Z * math.tan(math.rad(workspace.CurrentCamera.FieldOfView * 0.5)) * 2) * 100
                local width, height = math.floor(40 * scale_factor), math.floor(62 * scale_factor)

                if Visible then
                    Box.Size = Vector2.new(width, height)
                    Box.Position = Vector2.new(Pos.X - Box.Size.X / 2, Pos.Y - Box.Size.Y / 2)
                    Box.Visible = true
                else
                    Box.Visible = false
                end
            else
                Box:Destroy()
                Connection:Disconnect()
                BoxESP[Player] = nil
            end
        end)

        BoxESP[Player] = {
            Box = Box,
            Connection = Connection
        }
    end
end

local function updateESP()
    for _, Player in pairs(Players:GetPlayers()) do
        if Player.Character then
            BoxESP.Create(Player)
        end
    end
end

RunService.RenderStepped:Connect(updateESP)

-- Ensure ESP boxes for the local player are removed if needed
Players.PlayerRemoving:Connect(function(player)
    if BoxESP[player] then
        BoxESP[player].Box:Destroy()
        BoxESP[player].Connection:Disconnect()
        BoxESP[player] = nil
    end
end)

LocalPlayer.CharacterAdded:Connect(function()
    if BoxESP[LocalPlayer] then
        BoxESP[LocalPlayer].Box:Destroy()
        BoxESP[LocalPlayer].Connection:Disconnect()
        BoxESP[LocalPlayer] = nil
    end
end)
local function API_Check()
    if Drawing == nil then
        return "No"
    else
        return "Yes"
    end
end

local Find_Required = API_Check()

if Find_Required == "No" then
    game:GetService("StarterGui"):SetCore("SendNotification",{
        Title = "Exunys Developer";
        Text = "Crosshair script could not be loaded because your exploit is unsupported.";
        Duration = math.huge;
        Button1 = "OK"
    })

    return
end

local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")
local Camera = workspace.CurrentCamera

local Typing = false

local ViewportSize_ = Camera.ViewportSize / 2
local Axis_X, Axis_Y = ViewportSize_.X, ViewportSize_.Y

local HorizontalLine = Drawing.new("Line")
local VerticalLine = Drawing.new("Line")

_G.SendNotifications = true   -- If set to true then the script would notify you frequently on any changes applied and when loaded / errored. (If a game can detect this, it is recommended to set it to false)
_G.DefaultSettings = false   -- If set to true then the script would create a crosshair with the default settings regardless of any changes.
_G.ToMouse = false   -- If set to true then the crosshair will be positioned to your mouse cursor's position. If set to false it will be positioned to the center of your screen.

_G.CrosshairVisible = true   -- If set to true then the crosshair would be visible and vice versa.
_G.CrosshairSize = 20   -- The size of the crosshair.
_G.CrosshairThickness = 1   -- The thickness of the crosshair.
_G.CrosshairColor = Color3.fromRGB(228, 8, 10)   -- The color of the crosshair
_G.CrosshairTransparency = 1   -- The transparency of the crosshair.

_G.DisableKey = Enum.KeyCode.Q   -- The key that enables / disables the crosshair.

RunService.RenderStepped:Connect(function()
    local Real_Size = _G.CrosshairSize / 2

    HorizontalLine.Color = _G.CrosshairColor
    HorizontalLine.Thickness = _G.CrosshairThickness
    HorizontalLine.Visible = _G.CrosshairVisible
    HorizontalLine.Transparency = _G.CrosshairTransparency
    
    VerticalLine.Color = _G.CrosshairColor
    VerticalLine.Thickness = _G.CrosshairThickness
    VerticalLine.Visible = _G.CrosshairVisible
    VerticalLine.Transparency = _G.CrosshairTransparency
    
    if _G.ToMouse == true then
        HorizontalLine.From = Vector2.new(UserInputService:GetMouseLocation().X - Real_Size, UserInputService:GetMouseLocation().Y)
        HorizontalLine.To = Vector2.new(UserInputService:GetMouseLocation().X + Real_Size, UserInputService:GetMouseLocation().Y)
        
        VerticalLine.From = Vector2.new(UserInputService:GetMouseLocation().X, UserInputService:GetMouseLocation().Y - Real_Size)
        VerticalLine.To = Vector2.new(UserInputService:GetMouseLocation().X, UserInputService:GetMouseLocation().Y + Real_Size)
    elseif _G.ToMouse == false then
        HorizontalLine.From = Vector2.new(Axis_X - Real_Size, Axis_Y)
        HorizontalLine.To = Vector2.new(Axis_X + Real_Size, Axis_Y)
    
        VerticalLine.From = Vector2.new(Axis_X, Axis_Y - Real_Size)
        VerticalLine.To = Vector2.new(Axis_X, Axis_Y + Real_Size)
    end
end)

if _G.DefaultSettings == true then
    _G.CrosshairVisible = true
    _G.CrosshairSize = 25
    _G.CrosshairThickness = 1
    _G.CrosshairColor = Color3.fromRGB(228, 8, 10)
    _G.CrosshairTransparency = 0.15
    _G.DisableKey = Enum.KeyCode.Q
end

UserInputService.TextBoxFocused:Connect(function()
    Typing = true
end)

UserInputService.TextBoxFocusReleased:Connect(function()
    Typing = false
end)

UserInputService.InputBegan:Connect(function(Input)
    if Input.KeyCode == _G.DisableKey and Typing == false then
        _G.CrosshairVisible = not _G.CrosshairVisible
        
        if _G.SendNotifications == true then
            game:GetService("StarterGui"):SetCore("SendNotification",{
                Title = "Exunys Developer";
                Text = "The crosshair's visibility is now set to "..tostring(_G.CrosshairVisible)..".";
                Duration = 5;
            })
        end
    end
end)
