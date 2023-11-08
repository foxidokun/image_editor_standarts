```c++
// plugin.h

extern "C" Plugin* getInstance(App *);

enum class InterfaceType {
    Tool,
    Filter
};

struct Interface {};

struct Plugin {
    /* где-то тут лежит App*, но это дело автора плагина */
    uint64_t id;
    const char[] name;
    InterfaceType type;

    virtual Interface *getInstance() = 0;
    virtual ~Plugin() = 0;
}

struct Tool: public Interface {

}

struct Filter: public Interface {

}

struct App {
    GuiI *root;
    EventManagerI *event_manager; 
    ToolManagerI *tool_manager;
    FilterManagerI *filter_manager; 
}

struct GuiI {
    Vec2 getSize(); // размер доступной для рисования площади (которую можно запросить)

    /// @brief запросить RT.
    /// Идейно создает новое свое окно / отдает какое-то, абсолютно пустое, с единственным RT на все окно.
    /// @param size -- размер запрашиваемой области
    /// @param pos  -- (относительное) смещение запрашиваемой области
    virtual RenderTarget* getRenderTarget(Vec2 size, Vec2 pos, PLugin *self) = 0;

    /// @brief Создает окно с параметрами, каким-то образом узнает у пользователя 
    ///     значения параметров и потом возвращает их интерфейсу через Interface::set_params
    /// @note окно не обязательно модальное, да и вообще implementation defined. Мем в том, что плагин находится в 
    ///     неопределенном/дефолтном состоянии между createParamWindow и Interface::set_parans
    virtual void createParamWindow(Array<const char *> param_names, Interface * self) = 0;
}

struct EventManagerI {
    virtual void registerObject(EventProcessableI *object)   = 0;

    // 0 минимальный, ивенты приходят только объектам с их priority >= установленной по этому типу
    virtual void setPriority(EventType, uint8_t priority)   = 0;
    virtual void unregisterObject(EventProcessableI *object) = 0;
}

struct EventProcessableI {
    // MouseContext хранит в себе координаты относительно позиции RT из GuiI::getRenderTarget.
    // Мотивация: если RT это не весь экран, а RT в каком-то окне (как идейно и планировалось), то, 
    // строго говоря, плагин не знает где в реальном мире находится RT (его могли перетаскивать и проч)
    // и не может пересчитать их в локальные.
    
    /// @warning aka proposal: тогда вызов этих функций без предварительного вызова getRenderTarget UB.

    virtual bool onMouseMove(MouseContext context) = 0;
    virtual bool onMouseRelease(MouseContext context) = 0;
    virtual bool onMousePress(MouseContext context) = 0;
    virtual bool onKeyboardPress(KeyboardContext context) = 0;
    virtual bool onKeyboardRelease(KeyboardContext context) = 0;
    virtual bool onClock(... context) = 0;
}

/// @note см про относительность координат
struct MouseContext {
    Vec2 position;
    MouseButton button;
}

enum class MouseButton {
    Left,
    Right
}

struct KeyboardContext {
    bool alt;
    bool shift;
    bool ctrl;

    Key key; // Копипастим из SFML
}

```

0. Автор основной программы не долбоеб
1. В одном файле не более одного типа интерфейса. Либо Tool, либо Filter