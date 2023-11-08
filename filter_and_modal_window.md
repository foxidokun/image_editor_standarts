## Фильтры

```c++
class Filter {
    /* filter  */

    // Применяеь эффект, никаких окон не создает
    void do(RenderTarget &rt);

    // Возвращяет имена параметров (eg для blur фильтра будет радиус)
    virtual std::vector<const char *> get_param_names() = 0; 

    // Кто-то извне запросил параметры через окно настроек и возвращяет vector<double> параметров в том же порядке
    // что они и заданы в get_param_names
    virtual void set_params(const std::vector<double>& params) = 0;
}

// Прокидывается кнопкам фильтров (set_filter) и канвасам (для set_rt)
class FilterManager {
    Filter *last_filter;
    RenderTarget *rt;

    // Выставляется кнопками фильтров, которые применяют конкретный фильтр
    void set_filter(Filter* filter);
    
    // Для кнопки recent filter
    void get_filter(Filter* filter);

    // Выставляется канвасами, когда на них начинают рисовать.
    void set_rt(RenderTarget* rt);
}

// Примерный пример on_callback для Brush 
void brush_callback() {
    RenderTarget *rt = FilterManager.get_rt();
    Blur.do(rt);
    FilterManager.set_filter(Blur);
}
```

## Модальность

#### Бан ивентов

```c++

class EventProcessable {
    /* ... все, что было обсуждено ранее ... */
    uint8_t priority; // для EventManager::privatize_events
}

// В целом необходимый класс
//  Нужен для записи, воспроизведения, подмены и бана ивентов
class EventManager: public EventProcessable {
    // Тут лежит main window / window amanger / you name it, то есть корень всех виджетов. И логгер. 
    // Он не владеет детьми, то есть не вызывает деструктор при удалении
    std::list<EventProcessable *> childs;
    int priorities[EVENT_TYPES_NUM]; // приоритеты per type

    void register_object  (EventProcessable* ); // добавить сына
    void unregister_object(EventProcessable* ); // убить сына
    /* ... Event Processable API ... */
    // !Важно: EventManager игнорирует стопы (ребенок сказал, что обработал), чтобы работал условный логгер

    // Все события с типом из events передает listener'у и тем детям, чем приоритет больше либо равно, чем у listener. (то есть у логера типо максимальный приоритет).
    void privatize_events(vector<EventType /*enum-like type*/> events, int priority); // задает приоритет для этих типов

    void resert_prioritizes(); // сброс всех приоритетов до деффолтных (нуля)

}

class ModalWindow: public Window {
    EventManager *em;

    ModalWindow() {
        // 1. конструктор Window
        // 2. приватизация ивентов
    }

    ~ModalWindow() {
        // 1. разбан ивентов
        // 2. деструктор Window
    }
}
```