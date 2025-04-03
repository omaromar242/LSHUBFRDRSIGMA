--------------------------------------------------
-- DrRay UI Library – Full Example Script
--------------------------------------------------

-- Load the DrRay UI Library
local DrRayLibrary = loadstring(game:HttpGet("https://raw.githubusercontent.com/AZYsGithub/DrRay-UI-Library/main/DrRay.lua"))()

-- Create the UI window
local window = DrRayLibrary:Load("DrRay", "Default")

--------------------------------------------------
-- TAB 1: FEATURES (ESP, Hitbox, and ESP Books)
--------------------------------------------------
local featuresTab = DrRayLibrary.newTab("Features", "ImageIdHere")

-- Services & LocalPlayer reference
local Players = game:GetService("Players")
local LocalPlayer = Players.LocalPlayer

---------------------------
-- ESP Students/Teachers
---------------------------
local ESPEnabled = false

local function applyESP(player)
    if player == LocalPlayer then return end
    local character = player.Character
    if character and not character:FindFirstChild("FpeHighlight") then
        local highlight = Instance.new("Highlight")
        highlight.Name = "FpeHighlight"
        highlight.Adornee = character
        highlight.FillTransparency = 1       -- Outline only
        highlight.OutlineTransparency = 0    -- Fully visible outline
        highlight.OutlineColor = (player.TeamColor and player.TeamColor.Color) or Color3.new(1,1,1)
        highlight.DepthMode = Enum.HighlightDepthMode.AlwaysOnTop -- See through walls
        highlight.Parent = character
    end
end

local function removeESP(player)
    local character = player.Character
    if character then
        local h = character:FindFirstChild("FpeHighlight")
        if h then
            h:Destroy()
        end
    end
end

local function updateAllESP(enabled)
    for _, player in pairs(Players:GetPlayers()) do
        if enabled then
            applyESP(player)
        else
            removeESP(player)
        end
    end
end

local function connectCharacter(player)
    player.CharacterAdded:Connect(function(character)
        if ESPEnabled then
            wait(0.5)
            applyESP(player)
        end
    end)
end

for _, player in pairs(Players:GetPlayers()) do
    connectCharacter(player)
end
Players.PlayerAdded:Connect(connectCharacter)

featuresTab.newToggle("ESP Students/Teachers", "Toggle outline-only ESP (players)", false, function(state)
    ESPEnabled = state
    updateAllESP(ESPEnabled)
    print("[ESP] Students/Teachers is now:", state)
end)

---------------------------
-- Hitbox All
---------------------------
local HitboxEnabled = false
local hitboxLoopActive = false

local function updateHitboxes()
    for _, player in pairs(Players:GetPlayers()) do
        if player ~= LocalPlayer then
            local character = player.Character
            if character then
                local hrp = character:FindFirstChild("HumanoidRootPart")
                if hrp then
                    hrp.Size = Vector3.new(10, 10, 10)
                end
            end
        end
    end
end

featuresTab.newToggle("Hitbox All", "Every 2 sec, sets non-local players' HRP size to (10,10,10)", false, function(state)
    HitboxEnabled = state
    if HitboxEnabled and not hitboxLoopActive then
        hitboxLoopActive = true
        spawn(function()
            while hitboxLoopActive do
                updateHitboxes()
                wait(2)
            end
        end)
    elseif not HitboxEnabled then
        hitboxLoopActive = false
    end
    print("[Hitbox All] is now:", state)
end)

---------------------------
-- ESP Books
---------------------------
local ESPBooksEnabled = false
local espBooksLoopActive = false

local function applyESPBook(book)
    if book:FindFirstChild("FpeHighlight") then return end
    local highlight = Instance.new("Highlight")
    highlight.Name = "FpeHighlight"
    highlight.Adornee = book
    highlight.FillTransparency = 1       -- Outline only
    highlight.OutlineTransparency = 0    -- Fully visible outline
    highlight.OutlineColor = Color3.new(1, 1, 0)  -- Yellow for books
    highlight.DepthMode = Enum.HighlightDepthMode.AlwaysOnTop
    highlight.Parent = book
end

local function removeESPBook(book)
    if book:FindFirstChild("FpeHighlight") then
        book.FpeHighlight:Destroy()
    end
end

local function updateAllESPBooks(enabled)
    local booksFolder = workspace:FindFirstChild("Books")
    if not booksFolder then return end
    for _, book in pairs(booksFolder:GetChildren()) do
        if book.Name == "GetSmarterBook" then
            if enabled then
                applyESPBook(book)
            else
                removeESPBook(book)
            end
        end
    end
end

featuresTab.newToggle("ESP Books", "Toggle ESP for Books (GetSmarterBook)", false, function(state)
    ESPBooksEnabled = state
    if ESPBooksEnabled and not espBooksLoopActive then
        espBooksLoopActive = true
        spawn(function()
            while espBooksLoopActive do
                updateAllESPBooks(true)
                wait(2)
            end
        end)
    elseif not ESPBooksEnabled then
        espBooksLoopActive = false
        updateAllESPBooks(false)
    end
    print("[ESP Books] is now:", state)
end)

--------------------------------------------------
-- TAB 2: QUIZ (Questions & Answers as Static Text)
--------------------------------------------------
local quizTab = DrRayLibrary.newTab("Quiz", "ImageIdHere")

-- Define the list of questions and answers.
local questionsAndAnswers = {
    {"What is 2 - 3?", "-1"},
    {"What is 7 x 8?", "56"},
    {"What is 9 + 10?", "19"},
    {"What is 10 / 100?", "0.1"},
    {"What is 10 / 10?", "1"},
    {"What is π?", "≈ 3.1416"},
    {"Who was the 1st president of the United States?", "George Washington"},
    {"Which era marked a switch from agricultural practices to industrial practices?", "Industrial Revolution"},
    {"What ended in 1985?", "The Cold War's peak tensions, New Coke, etc. (Need more context)"},
    {"When was Hershey's founded?", "1894"},
    {"When did World War 1 start?", "1914"},
    {"What is the powerhouse of a cell?", "Mitochondria"},
    {"What is when a tadpole transforms into a frog?", "Metamorphosis"},
    {"All living things are made up of cells", "True"},
    {"What is when a plant uses sunlight to grow?", "Photosynthesis"},
    {"What is the largest mammal on Earth?", "Blue Whale"},
    {"Which isn't a characteristic of life?", "Many possible answers (e.g., 'Inability to adapt')"},
    {"Which symbol represents silver?", "Ag"},
    {"At what temperature does water freeze (In Fahrenheit)?", "32°F"},
    {"Which of the following is not a note in the musical scale?", "Example: 'H' (since standard scales use A-G)"},
    {"A Treble clef is used for illustrating notes that are:", "Higher-pitched notes"},
    {"A staff consists of how many horizontal lines?", "5"},
    {"How many notes are in the scale?", "Typically 7 (or 8 if including octave)"},
    {"Which of the following markings would affect the length of a note?", "Fermata, Dotted notes, etc."},
    {"What is Brazil's official language?", "Portuguese"},
    {"Which language has the most native speakers in the world?", "Chinese (Mandarin)"},
    {"What is Money called in Spanish?", "Dinero"},
    {"What is an example of a Metaphor?", "\"Time is a thief\""},
    {"Which colors make up Orange?", "Red + Yellow"},
    {"Which one of the following is a primary color?", "Red, Blue, or Yellow (depending on color model)"}
}

-- For each quiz entry, add a static text label for the question and below it a smaller text label for the answer.
for i, qa in ipairs(questionsAndAnswers) do
    local questionText = qa[1]
    local answerText = qa[2]
    
    -- Create a label for the question.
    if quizTab.newLabel then
        quizTab.newLabel("Q" .. i .. ": " .. questionText)
    else
        -- Fallback: use newButton as a non-clickable label if newLabel isn't available.
        local lbl = quizTab.newButton("Q" .. i .. ": " .. questionText, "", function() end)
        lbl:SetEnabled(false)
    end
    
    -- Create a smaller label for the answer.
    if quizTab.newLabel then
        quizTab.newLabel("A" .. i .. ": " .. answerText)
    else
        local ansLbl = quizTab.newButton("A" .. i .. ": " .. answerText, "", function() end)
        ansLbl:SetEnabled(false)
    end
end

--------------------------------------------------
-- Optional: Built-in UI functions from DrRay Library:
-- window:Toggle(), window:Open(), window:Close(), window:Hide(), window:Show()
--------------------------------------------------

-- End of script
