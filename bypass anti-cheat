local Encrypt = function(str, key)
    local byte = string.byte
    local sub = string.sub
    local char = string.char

    local endStr = {}

    for i = 1, #str do
        local keyPos = (i % #key) + 1
        endStr[i] = char(((byte(sub(str, i, i)) + byte(sub(key, keyPos, keyPos)))%126) + 1)
    end

    endStr = table.concat(endStr)
    return endStr
end

local Require = function(received)
    local client_loader = getcallingscript()
    local client_values = received.Parent.Special
    local client_remote = string.split(client_values.Value, "\\")[1]
    local client_encode = string.split(client_values.Value, "\\")[2]
    local client_module = received
    local client_signal = Instance.new("BindableEvent")
    local client_handle = nil

    local client_args = {
        Mode = "Fire",
        Module = client_module,
        Loader = client_loader,
        Sent = 0,
        Received = 0
    }

    client_remote = game:FindFirstChild(client_remote, true)

    client_handle = client_remote.OnClientEvent:Connect(function(_, name, key)
        if string.find(name, "GIVE_KEY") then
            client_handle:Disconnect()
            client_handle = nil
            client_signal:Fire(key)
            client_signal:Destroy()
            client_signal = nil
        end
    end)

    client_remote:FireServer(client_args, client_encode .. "GET_KEY")

    client_signal.Event:Once(function(client_key)
        local client_code = Encrypt("ClientCheck", client_key)

        while true do
            client_args.Sent = client_args.Sent + 1

            local args = {
                [1] = client_args,
                [2] = client_code,
                [3] = {
                    Received = client_args.Received,
                    Sent = client_args.Sent - 1
                },
                [4] = client_encode,
                -- [5] = math.random()
            }

            client_remote:FireServer(unpack(args))
            task.wait(3)
        end
    end)
end


local old
old = hookfunction(getrenv().require, function(module, ...)
    if module.Name == "Client" and module.Parent:FindFirstChild("Special") then
        task.spawn(Require, module)
        return coroutine.yield()
    end
    return old(module, ...)
end)
