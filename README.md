#include <iostream>
#include <string>
#include <functional>
#include <unordered_map>
#include <vector>
#include <cmath>

class EnumKeyCode {
public:
    static const int ButtonStart = 1;
    static const int RightShift = 2;
    static const int LeftShift = 3;
    static const int ButtonR2 = 4;
};

class InputObject {
public:
    int KeyCode;
    InputObject(int keyCode) : KeyCode(keyCode) {}
};

class Vector3 {
public:
    float x, y, z;
    Vector3(float x_=0, float y_=0, float z_=0) : x(x_), y(y_), z(z_) {}
    float Magnitude() const {
        return std::sqrt(x*x + y*y + z*z);
    }
    Vector3 Unit() const {
        float mag = Magnitude();
        if (mag == 0) return Vector3(0,0,0);
        return Vector3(x/mag, y/mag, z/mag);
    }
    Vector3 operator*(float scalar) const {
        return Vector3(x*scalar, y*scalar, z*scalar);
    }
    Vector3 operator+(const Vector3& other) const {
        return Vector3(x+other.x, y+other.y, z+other.z);
    }
};

class CFrame {
public:
    Vector3 p;
    CFrame(Vector3 pos = Vector3()) : po(pos) {}
    CFrame operator+(const Vector3& offset) const {
        return CFrame(p + offset);
    }
};

class Humanoid {
public:
    Vector3 MoveDirection;
    Humanoid(Vector3 moveDir = Vector3()) : MoveDirection(moveDir) {}
};

class Character {
public:
    CFrame HumanoidRootPart;
    Humanoid humanoid;
    Character() : HumanoidRootPart(CFrame()), humanoid(Humanoid()) {}
};

class Player {
public:
    Character* CharacterPtr;
    Player() : CharacterPtr(nullptr) {}
};

class RayfieldClass {
public:
    void Notify(const std::unordered_map<std::string, std::string>& params) {
        std::cout << "Notify: " << params.at("Title") << " - " << params.at("Content") << std::endl;
    }
    void Toggle(bool visible) {
        std::cout << "UI visibility toggled to " << (visible ? "true" : "false") << std::endl;
    }
};

class WindowClass {
public:
    WindowClass(const std::string& name) { (void)name; }
    void CreateTab(const std::string& name, int imageId) {
        (void)name; (void)imageId;
    }
};

class TabClass {
public:
    void CreateToggle(const std::string& name, bool currentValue, const std::function<void(bool)>& callback) {
        (void)name; (void)currentValue; (void)callback;
    }
    void CreateSlider(const std::string& name, float minRange, float maxRange, float increment, float currentValue, const std::function<void(float)>& callback) {
        (void)name; (void)minRange; (void)maxRange; (void)increment; (void)currentValue; (void)callback;
    }
}
  void SetKeyBind(const std::string& const inputobj& input, bool gpe, string key)

bool spe = false;
float sp = 0.03f;
bool sprintActive = false;
bool uiVisible = true;

RayfieldClass Rayfield;
Player player;
WindowClass* Window = nullptr;
TabClass* Tab = nullptr;

void OnInputBegan(const InputObject& input, bool gameProcessed) {
    if (gameProcessed) return;
    if (input.KeyCode == EnumKeyCode::ButtonStart) {

        std::cout << "VirtualInputManager: SendKeyEvent(true, RightShift)" << std::endl;
        std::cout << "VirtualInputManager: SendKeyEvent(false, RightShift)" << std::endl;
    }
    if (input.KeyCode == EnumKeyCode::RightShift) {
        uiVisible = !uiVisible;
        Rayfield.Toggle(uiVisible);
    }
}

void OnInputBeganSimple(const InputObject& input) {
    if (input.KeyCode == EnumKeyCode::LeftShift || input.KeyCode == EnumKeyCode::ButtonR2) {
        sprintActive = true;
    }
}

void OnInputEnded(const InputObject& input) {
    if (input.KeyCode == EnumKeyCode::LeftShift || input.KeyCode == EnumKeyCode::ButtonR2) {
        sprintActive = false;
    }
}

void OnHeartbeat() {
    if (!player.CharacterPtr) return;
    Character* character = player.CharacterPtr;
    Humanoid& humanoid = character->humanoid;
    CFrame& rootPart = character->HumanoidRootPart;

    if (spe && sprintActive && humanoid.MoveDirection.Magnitude() > 0) {
        Vector3 offset = humanoid.MoveDirection.Unit() * sp;
        rootPart = rootPart + offset;
        std::cout << "Character moved to position (" << rootPart.position.x << ", " << rootPart.position.y << ", " << rootPart.position.z << ")\n";
    }
}

int main() {
    Rayfield.Notify({
        {"Title", "intro"},
        {"Content", "sybau and dig in yo buh"},
        {"Duration", "2"},
        {"Image", "4483362458"}
    });

    Window = new WindowClass("idk a window");
    Tab = new TabClass();

    auto toggleCallback = [](bool value) { spe = value; };
    auto sliderCallback = [](float value) { sp = value; };

    Tab->CreateToggle("Toggle Sprint", false, toggleCallback);
    Tab->CreateSlider("Sprint Speed", 0.01f, 0.30f, 0.01f, 0.03f, sliderCallback);

    Character character;
    player.CharacterPtr = &character;

    OnInputBegan(InputObject(EnumKeyCode::ButtonStart), false);
    OnInputBegan(InputObject(EnumKeyCode::RightShift), false);
    OnInputBeganSimple(InputObject(EnumKeyCode::LeftShift));
    character.humanoid.MoveDirection = Vector3(1, 0, 0); 

    for (int i = 0; i < 5; ++i) {
        OnHeartbeat();
    }

    OnInputEnded(InputObject(EnumKeyCode::LeftShift));

    delete Window;
    delete Tab;

    return 0;
}
