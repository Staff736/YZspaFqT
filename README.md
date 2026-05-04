local RS = game:GetService("ReplicatedStorage")
local PlayerTurn = RS:WaitForChild("PlayerTurnInput")

task.spawn(function()
    while task.wait(0.2) do
        local enemy = workspace:FindFirstChild("Living") and workspace.Living:FindFirstChild("Test")
        
        if enemy then
            pcall(function()
                PlayerTurn:InvokeServer(
                    "Attack",
                    "Strike",
                    {
                        ["Attacking"] = enemy
                    }
                )
            end)
        end
    end
end)
