# Создание пользовательского интерфейса

|                      |                                                               |
| -------------------- | ------------------------------------------------------------- |
| **Тема**             | Построение интерфейса приложения средствами XAML              |
| **Среда разработки** | JetBrains Rider                                               |
| **Платформа**        | .NET MAUI                                                     |
| **Язык**             | C# / XAML                                                     |
| **Проект**           | Task Manager (главный экран)                                  |
| **Время выполнения** | 4 академических часа (базовая часть) + дополнительное задание |

---

## 1. Цель работы

Научиться создавать экран мобильного приложения с использованием основных элементов управления .NET MAUI и разметки XAML.

## 2. Задачи

1. Создать проект .NET MAUI в среде Rider.
2. Описать модель данных `TaskItem`.
3. Построить разметку экрана с помощью `Grid`, `VerticalStackLayout`, `HorizontalStackLayout`.
4. Использовать элементы `Entry`, `Button`, `Switch`, `CheckBox`, `CollectionView`.
5. Создать шаблон элемента списка и визуально выделить выполненные задачи.
6. Проверить поведение интерфейса на экране небольшого размера.

## 3. Краткие теоретические сведения

**XAML** — декларативный язык разметки, на котором описывается внешний вид страниц. Каждая страница состоит из файла разметки (`MainPage.xaml`) и файла логики (`MainPage.xaml.cs`).

| Элемент | Назначение |
|---|---|
| `Grid` | Контейнер-таблица. Размеры строк и столбцов: `Auto` (по содержимому), `*` (оставшееся место), число (фиксированный размер). |
| `VerticalStackLayout` | Располагает дочерние элементы друг под другом. |
| `HorizontalStackLayout` | Располагает дочерние элементы в ряд. |
| `Entry` | Однострочное поле ввода текста. |
| `Button` | Кнопка, событие `Clicked`. |
| `Switch` | Переключатель «вкл/выкл», событие `Toggled`. |
| `CheckBox` | Флажок, свойство `IsChecked`. |
| `CollectionView` | Прокручиваемый список, элементы которого создаются по шаблону `DataTemplate`. |
| `DataTrigger` | Меняет свойства элемента при выполнении условия привязки. |

**Привязка данных (Binding)** связывает свойство элемента интерфейса со свойством объекта: `Text="{Binding Title}"`. Чтобы интерфейс реагировал на изменение свойства во время работы, класс должен реализовывать интерфейс `INotifyPropertyChanged`.

---

## 4. Ход работы
---

### Шаг 1. Создание модели

1. В **Solution Explorer** щёлкните правой кнопкой по проекту `TaskManager` → **Add → New Folder**, назовите папку `Models`.
2. Щёлкните правой кнопкой по папке `Models` → **Add → Class/Interface**, введите имя `TaskItem`.
3. Замените содержимое файла:

```csharp
namespace TaskManager.Models;

public class TaskItem
{
    public int Id { get; set; }
    public string Title { get; set; } = string.Empty;
    public bool IsCompleted { get; set; }
}
```

> Это базовая модель из задания. На шаге 4 она будет доработана, чтобы вид задачи менялся сразу после установки флажка.


---

### Шаг 2. Разметка главного экрана

Откройте `MainPage.xaml` и **полностью замените** содержимое:

```xml
<?xml version="1.0" encoding="utf-8" ?>
<ContentPage xmlns="http://schemas.microsoft.com/dotnet/2021/maui"
             xmlns:x="http://schemas.microsoft.com/winfx/2009/xaml"
             xmlns:models="clr-namespace:TaskManager.Models"
             x:Class="TaskManager.MainPage"
             Shell.NavBarIsVisible="False">

    <Grid RowDefinitions="Auto,Auto,*"
          RowSpacing="12"
          Padding="16">

        <!-- Строка 0: заголовок -->
        <VerticalStackLayout Grid.Row="0" Spacing="2">
            <Label Text="Task Manager"
                   FontSize="28"
                   FontAttributes="Bold" />
            <Label Text="Список задач"
                   FontSize="14"
                   TextColor="Gray" />
        </VerticalStackLayout>

        <!-- Строка 1: ввод, кнопка, переключатель -->
        <VerticalStackLayout Grid.Row="1" Spacing="8">

            <Grid ColumnDefinitions="*,Auto" ColumnSpacing="8">
                <Entry x:Name="TaskEntry"
                       Grid.Column="0"
                       Placeholder="Введите задачу"
                       ReturnType="Done"
                       Completed="OnAddClicked" />
                <Button Grid.Column="1"
                        Text="Добавить"
                        Clicked="OnAddClicked" />
            </Grid>

            <HorizontalStackLayout Spacing="8">
                <Switch x:Name="HideCompletedSwitch"
                        Toggled="OnHideCompletedToggled" />
                <Label Text="Скрыть выполненные"
                       VerticalOptions="Center" />
            </HorizontalStackLayout>

        </VerticalStackLayout>

        <!-- Строка 2: список (одновременно область прокрутки) -->
        <CollectionView x:Name="TasksView"
                        Grid.Row="2"
                        SelectionMode="None">
            <!-- Шаблон элемента добавляется на шаге 4 -->
        </CollectionView>

    </Grid>
</ContentPage>
```

**Пояснения к разметке**

- Корневой `Grid` имеет три строки: две по содержимому (`Auto`) и одну, занимающую всё оставшееся место (`*`). Благодаря этому заголовок и форма ввода остаются на месте, а список прокручивается.
- `CollectionView` сам обеспечивает вертикальную прокрутку и является «областью прокрутки» экрана.
- Обработчики `OnAddClicked` и `OnHideCompletedToggled` будут созданы на шаге 3.

> **Важно.** Не помещайте вертикальный `CollectionView` внутрь вертикального `ScrollView`: список потеряет виртуализацию и будет работать медленно.

---

### Шаг 3. Тестовые данные и логика страницы

Откройте `MainPage.xaml.cs` (раскройте `MainPage.xaml` в дереве проекта) и **замените** содержимое:

```csharp
using TaskManager.Models;

namespace TaskManager;

public partial class MainPage : ContentPage
{
    private readonly List<TaskItem> _tasks = new();
    private int _nextId = 1;

    public MainPage()
    {
        InitializeComponent();
        LoadTestData();
        RefreshList();
    }

    // Временная коллекция тестовых задач
    private void LoadTestData()
    {
        string[] titles =
        {
            "Изучить XAML",
            "Выполнить практическую работу",
            "Повторить C#",
            "Подготовить проект"
        };

        foreach (var title in titles)
        {
            _tasks.Add(new TaskItem { Id = _nextId++, Title = title });
        }

        // Одна задача выполнена, чтобы сразу увидеть визуальное отличие
        _tasks[0].IsCompleted = true;
    }

    // Обновление списка с учётом переключателя
    private void RefreshList()
    {
        TasksView.ItemsSource = HideCompletedSwitch.IsToggled
            ? _tasks.Where(t => !t.IsCompleted).ToList()
            : _tasks.ToList();
    }

    // Добавление новой задачи
    private void OnAddClicked(object? sender, EventArgs e)
    {
        var title = TaskEntry.Text?.Trim();
        if (string.IsNullOrEmpty(title))
            return;

        _tasks.Add(new TaskItem { Id = _nextId++, Title = title });
        TaskEntry.Text = string.Empty;
        RefreshList();
    }

    // Переключатель «Скрыть выполненные»
    private void OnHideCompletedToggled(object? sender, ToggledEventArgs e)
    {
        RefreshList();
    }
}
```

**Пояснения**

- `LoadTestData` создаёт временную коллекцию из четырёх задач.
- `RefreshList` присваивает список свойству `ItemsSource`. Если переключатель включён, выполненные задачи отфильтровываются.
- `OnAddClicked` проверяет, что строка не пустая, добавляет задачу и очищает поле ввода.

---

### Шаг 4. Шаблон отображения задачи в CollectionView

#### 4.1. Доработка модели

Чтобы при установке флажка `IsCompleted` интерфейс перерисовывался, реализуем `INotifyPropertyChanged`. Обновите `Models/TaskItem.cs`:

```csharp
using System.ComponentModel;
using System.Runtime.CompilerServices;

namespace TaskManager.Models;

public class TaskItem : INotifyPropertyChanged
{
    private bool _isCompleted;

    public int Id { get; set; }
    public string Title { get; set; } = string.Empty;

    public bool IsCompleted
    {
        get => _isCompleted;
        set
        {
            if (_isCompleted == value) return;
            _isCompleted = value;
            OnPropertyChanged();
        }
    }

    public event PropertyChangedEventHandler? PropertyChanged;

    protected void OnPropertyChanged([CallerMemberName] string? name = null)
        => PropertyChanged?.Invoke(this, new PropertyChangedEventArgs(name));
}
```

#### 4.2. Шаблон элемента

В `MainPage.xaml` замените комментарий внутри `CollectionView` на шаблон:

```xml
<CollectionView x:Name="TasksView"
                Grid.Row="2"
                SelectionMode="None">

    <CollectionView.ItemTemplate>
        <DataTemplate x:DataType="models:TaskItem">
            <Border Margin="0,4"
                    Padding="10,6"
                    Stroke="Gray"
                    StrokeThickness="1"
                    StrokeShape="RoundRectangle 8">

                <Grid ColumnDefinitions="Auto,*" ColumnSpacing="8">

                    <CheckBox Grid.Column="0"
                              IsChecked="{Binding IsCompleted}"
                              VerticalOptions="Center" />

                    <Label Grid.Column="1"
                           Text="{Binding Title}"
                           FontSize="16"
                           VerticalOptions="Center"
                           LineBreakMode="WordWrap">
                        <Label.Triggers>
                            <DataTrigger TargetType="Label"
                                         Binding="{Binding IsCompleted}"
                                         Value="True">
                                <Setter Property="TextDecorations" Value="Strikethrough" />
                                <Setter Property="TextColor" Value="Gray" />
                            </DataTrigger>
                        </Label.Triggers>
                    </Label>

                </Grid>
            </Border>
        </DataTemplate>
    </CollectionView.ItemTemplate>

</CollectionView>
```

**Пояснения**

- `x:DataType="models:TaskItem"` включает компилируемые привязки: они быстрее, а Rider подсказывает имена свойств.
- `CheckBox` с привязкой `IsChecked` даёт вид `[ ] Изучить XAML`.
- `DataTrigger` создаёт визуальное отличие выполненной задачи: текст зачёркивается и становится серым.
- Колонки `Auto,*` позволяют длинному тексту переноситься на новую строку и не выходить за границы экрана.

#### 4.3. Запуск и проверка

Нажмите **Run** (зелёный треугольник). Убедитесь, что:

- отображаются четыре задачи, первая зачёркнута;
- при установке флажка текст зачёркивается сразу;
- кнопка **Добавить** и клавиша Enter добавляют задачу;
- пустая строка не добавляется;
- переключатель **Скрыть выполненные** скрывает выполненные задачи.

---

### Шаг 5. Проверка на экране небольшого размера

1. Запустите приложение на эмуляторе с маленьким экраном (Pixel 2 или аналогичный).
2. Дополнительно (по желанию) запустите Windows-версию командой и уменьшайте ширину окна:

   ```bash
   dotnet build -t:Run -f net8.0-windows10.0.19041.0
   ```

   Укажите значение `TargetFramework` из вашего `.csproj`.

3. Выполните проверки и запишите результаты в отчёт:

| № | Проверка | Ожидаемый результат | Фактический результат |
|---|---|---|---|
| 1 | Добавить задачу длиной 80–100 символов | Текст переносится, не обрезается | |
| 2 | Добавить 15–20 задач | Список прокручивается, заголовок и поле ввода остаются на месте | |
| 3 | Повернуть эмулятор в альбомную ориентацию | Элементы не перекрываются, кнопка видна | |
| 4 | Нажать **Добавить** при пустом поле | Задача не добавляется | |
| 5 | Включить **Скрыть выполненные** | Выполненные задачи исчезают | |

**Если элементы не помещаются:**

- сократите текст кнопки (`Text="+"`);
- задайте кнопке `MinimumWidthRequest`;
- уменьшите `Padding` корневого `Grid`.

---

## 5. Дополнительное задание: дата, приоритет, цветовая индикация

### 5.1. Перечисление приоритета

Создайте файл `Models/TaskPriority.cs`:

```csharp
namespace TaskManager.Models;

public enum TaskPriority
{
    Low,
    Medium,
    High
}
```

### 5.2. Итоговая модель

Обновите `Models/TaskItem.cs`:

```csharp
using System.ComponentModel;
using System.Runtime.CompilerServices;
using Microsoft.Maui.Graphics;

namespace TaskManager.Models;

public class TaskItem : INotifyPropertyChanged
{
    private bool _isCompleted;

    public int Id { get; set; }
    public string Title { get; set; } = string.Empty;
    public DateTime CreatedAt { get; set; } = DateTime.Now;
    public TaskPriority Priority { get; set; } = TaskPriority.Medium;

    public bool IsCompleted
    {
        get => _isCompleted;
        set
        {
            if (_isCompleted == value) return;
            _isCompleted = value;
            OnPropertyChanged();
        }
    }

    // Цвет индикатора приоритета
    public Color PriorityColor => Priority switch
    {
        TaskPriority.High => Colors.Crimson,
        TaskPriority.Medium => Colors.Orange,
        _ => Colors.SeaGreen
    };

    // Текст приоритета
    public string PriorityText => Priority switch
    {
        TaskPriority.High => "Высокий",
        TaskPriority.Medium => "Средний",
        _ => "Низкий"
    };

    public event PropertyChangedEventHandler? PropertyChanged;

    protected void OnPropertyChanged([CallerMemberName] string? name = null)
        => PropertyChanged?.Invoke(this, new PropertyChangedEventArgs(name));
}
```

> Хранение цвета в модели упрощает лабораторную работу. В профессиональных проектах цвет выносят в `IValueConverter`, чтобы не смешивать данные и интерфейс. Это тема следующих работ.

### 5.3. Выбор приоритета в форме

В `MainPage.xaml`, в блоке `VerticalStackLayout Grid.Row="1"`, добавьте `Picker` **после** `Grid` с полем ввода:

```xml
<Picker x:Name="PriorityPicker"
        Title="Приоритет" />
```

### 5.4. Шаблон с датой и цветовой полосой

Замените `DataTemplate` целиком:

```xml
<DataTemplate x:DataType="models:TaskItem">
    <Border Margin="0,4"
            Padding="0"
            Stroke="Gray"
            StrokeThickness="1"
            StrokeShape="RoundRectangle 8">

        <Grid ColumnDefinitions="6,Auto,*" ColumnSpacing="8">

            <!-- Цветовая индикация приоритета -->
            <BoxView Grid.Column="0"
                     Color="{Binding PriorityColor}" />

            <CheckBox Grid.Column="1"
                      IsChecked="{Binding IsCompleted}"
                      VerticalOptions="Center" />

            <VerticalStackLayout Grid.Column="2"
                                 Padding="0,8,8,8"
                                 Spacing="2">

                <Label Text="{Binding Title}"
                       FontSize="16"
                       LineBreakMode="WordWrap">
                    <Label.Triggers>
                        <DataTrigger TargetType="Label"
                                     Binding="{Binding IsCompleted}"
                                     Value="True">
                            <Setter Property="TextDecorations" Value="Strikethrough" />
                            <Setter Property="TextColor" Value="Gray" />
                        </DataTrigger>
                    </Label.Triggers>
                </Label>

                <HorizontalStackLayout Spacing="8">
                    <Label Text="{Binding PriorityText, StringFormat='Приоритет: {0}'}"
                           FontSize="12"
                           TextColor="{Binding PriorityColor}" />
                    <Label Text="{Binding CreatedAt, StringFormat='{0:dd.MM.yyyy HH:mm}'}"
                           FontSize="12"
                           TextColor="Gray" />
                </HorizontalStackLayout>

            </VerticalStackLayout>
        </Grid>
    </Border>
</DataTemplate>
```

### 5.5. Изменения в `MainPage.xaml.cs`

**а) Конструктор** — заполните `Picker` до вызова `LoadTestData()`:

```csharp
public MainPage()
{
    InitializeComponent();

    PriorityPicker.ItemsSource = new List<string> { "Низкий", "Средний", "Высокий" };
    PriorityPicker.SelectedIndex = 1;   // по умолчанию «Средний»

    LoadTestData();
    RefreshList();
}
```

**б) Тестовые данные:**

```csharp
private void LoadTestData()
{
    _tasks.Add(new TaskItem { Id = _nextId++, Title = "Изучить XAML",
        Priority = TaskPriority.Low, CreatedAt = DateTime.Now.AddDays(-2), IsCompleted = true });
    _tasks.Add(new TaskItem { Id = _nextId++, Title = "Выполнить практическую работу",
        Priority = TaskPriority.High, CreatedAt = DateTime.Now });
    _tasks.Add(new TaskItem { Id = _nextId++, Title = "Повторить C#",
        Priority = TaskPriority.Medium, CreatedAt = DateTime.Now.AddDays(-1) });
    _tasks.Add(new TaskItem { Id = _nextId++, Title = "Подготовить проект",
        Priority = TaskPriority.Low, CreatedAt = DateTime.Now });
}
```

**в) Добавление задачи:**

```csharp
private void OnAddClicked(object? sender, EventArgs e)
{
    var title = TaskEntry.Text?.Trim();
    if (string.IsNullOrEmpty(title))
        return;

    _tasks.Add(new TaskItem
    {
        Id = _nextId++,
        Title = title,
        Priority = (TaskPriority)PriorityPicker.SelectedIndex,
        CreatedAt = DateTime.Now
    });

    TaskEntry.Text = string.Empty;
    RefreshList();
}
```

### 5.6. Проверка

Запустите приложение. Слева у каждой задачи должна быть цветная полоса: красная у высокого приоритета, оранжевая у среднего, зелёная у низкого. Под названием отображаются приоритет и дата создания.

![Рис. 9 — Экран с датой, приоритетом и цветовой индикацией](images/fig9.png)

---

## 6. Типичные ошибки и их устранение

| Симптом | Причина и решение |
|---|---|
| `The name 'InitializeComponent' does not exist` | Не совпадают `x:Class` в XAML и namespace в `.cs`. Оба должны указывать на `TaskManager.MainPage`. |
| Ошибка в строке `xmlns:models` | Namespace модели должен быть `TaskManager.Models`. Проверьте строку `namespace` в `TaskItem.cs`. |
| Флажок ставится, а текст не зачёркивается | Модель не реализует `INotifyPropertyChanged` (см. шаг 4.1). |
| Список пуст | `RefreshList()` не вызван или вызван до `InitializeComponent()`. |
| Приложение не собирается под Android | Не установлен Android SDK или не создан эмулятор. Проверьте **Settings → Build, Execution, Deployment → Android**. |
| Предупреждения о привязках | Не указан `x:DataType` в `DataTemplate` или неверно записан namespace `models`. |

---

## 7. Содержание отчёта

1. Титульный лист, тема, цель работы.
2. Скриншоты (рис. 1–9) с подписями.
3. Листинги: `TaskItem.cs`, `TaskPriority.cs`, `MainPage.xaml`, `MainPage.xaml.cs`.
4. Таблица результатов проверки на маленьком экране (шаг 5).
5. Ответы на контрольные вопросы.
6. Вывод.

## 8. Контрольные вопросы

1. Чем `VerticalStackLayout` отличается от `Grid`? Когда лучше применять каждый из них?
2. Что означают значения `Auto`, `*` и число в `RowDefinitions` / `ColumnDefinitions`?
3. Почему для списков предпочтителен `CollectionView`, а не `StackLayout` внутри `ScrollView`?
4. Для чего нужен `DataTemplate`?
5. Как работает `DataTrigger`? Приведите пример.
6. Зачем модели реализовывать интерфейс `INotifyPropertyChanged`?
7. Чем `Switch` отличается от `CheckBox`?
8. Какие свойства помогают обеспечить корректное отображение на экранах разного размера?

## 9. Список источников

1. Документация .NET MAUI — Microsoft Learn: <https://learn.microsoft.com/dotnet/maui/>
2. Раздел «Layouts» и «CollectionView» документации .NET MAUI.
3. Документация JetBrains Rider — .NET MAUI: <https://www.jetbrains.com/help/rider/>




# Критерии оценки. Практическая работа №2

**Тема:** Создание пользовательского интерфейса (Task Manager, .NET MAUI, XAML)

Максимальный балл: **100** (базовая часть 85 + дополнительное задание 15).

---

## 1. Оценочная таблица

| №   | Критерий                              | Баллы   | Что проверяется                                                                                                                               |
| --- | ------------------------------------- | ------- | --------------------------------------------------------------------------------------------------------------------------------------------- |
| 1   | Создание проекта и структура          | 5       | Проект `TaskManager` создан в Rider, запускается на эмуляторе, папка `Models` создана                                                         |
| 2   | Модель `TaskItem`                     | 10      | Свойства `Id`, `Title`, `IsCompleted`; корректный namespace; реализован `INotifyPropertyChanged`                                              |
| 3   | Разметка экрана (XAML)                | 20      | Использованы `Grid`, `VerticalStackLayout`, `HorizontalStackLayout`; есть заголовок, `Entry`, `Button`, `Switch`; корректные `RowDefinitions` |
| 4   | Тестовые данные                       | 5       | Временная коллекция из 3–5 задач, минимум одна выполнена                                                                                      |
| 5   | `CollectionView` и шаблон             | 20      | `DataTemplate` с `CheckBox` и названием; привязки `{Binding}`; указан `x:DataType`                                                            |
| 6   | Визуальное отличие выполненной задачи | 10      | `DataTrigger` (зачёркивание, цвет); эффект виден сразу при установке флажка                                                                   |
| 7   | Логика страницы                       | 10      | Добавление задачи, проверка пустой строки, очистка поля, работа переключателя                                                                 |
| 8   | Проверка на маленьком экране          | 5       | Выполнена проверка, заполнена таблица результатов, есть скриншоты                                                                             |
| 9   | Отчёт и оформление                    | 10      | Полнота, скриншоты, листинги, вывод, ответы на вопросы                                                                                        |
|     | **Итого (базовая часть)**             | **95**  |                                                                                                                                               |
| 10  | Дополнительное задание                | 15      | Дата создания (4), приоритет и `Picker` (5), цветовая индикация (4), корректная работа при добавлении (2)                                     |
|     | **Максимум**                          | **110** | Баллы свыше 100 не переводятся в оценку, но повышают итог при пограничном значении                                                            |


---

## 2. Перевод баллов в оценку

| Баллы | Оценка | Уровень |
|---|---|---|
| 90–100 | **5 (отлично)** | Все требования выполнены, работа оформлена аккуратно, студент уверенно отвечает на вопросы |
| 75–89 | **4 (хорошо)** | Есть незначительные недочёты в коде или отчёте |
| 60–74 | **3 (удовлетворительно)** | Основная часть выполнена, есть существенные замечания |
| менее 60 | **2 (неудовлетворительно)** | Приложение не запускается или не выполнены обязательные элементы |

---

## 3. Подробные критерии по уровням

### 3.1. Разметка экрана (20 баллов)

| Баллы | Описание |
|---|---|
| 17–20 | Все обязательные элементы на месте, корректная структура `Grid` (`Auto,Auto,*`), аккуратные отступы, экран выглядит цельно |
| 12–16 | Элементы присутствуют, но структура не оптимальна (например, лишняя вложенность или отсутствует `HorizontalStackLayout`) |
| 6–11 | Отсутствует один из обязательных элементов или экран собран без `Grid` |
| 0–5 | Разметка не соответствует заданию или не компилируется |

### 3.2. `CollectionView` и шаблон (20 баллов)

| Баллы | Описание |
|---|---|
| 17–20 | Шаблон полностью соответствует заданию (`[ ] Название`), привязки корректны, использован `x:DataType`, список прокручивается |
| 12–16 | Шаблон работает, но нет `x:DataType` или есть замечания по оформлению |
| 6–11 | Список отображается без шаблона либо привязки работают частично |
| 0–5 | Список не отображается |

### 3.3. Визуальное отличие выполненной задачи (10 баллов)

| Баллы | Описание |
|---|---|
| 9–10 | Отличие реализовано через `DataTrigger`, обновляется сразу, применены минимум два свойства (зачёркивание и цвет) |
| 6–8 | Отличие есть, но применено одно свойство |
| 3–5 | Отличие работает только после перезагрузки списка (не реализован `INotifyPropertyChanged`) |
| 0–2 | Отличия нет |

### 3.4. Отчёт (10 баллов)

| Баллы | Описание |
|---|---|
| 9–10 | Есть все разделы: скриншоты, листинги, таблица проверки, ответы на вопросы, вывод |
| 6–8 | Отсутствует один-два элемента |
| 3–5 | Отчёт неполный, скриншоты не подписаны |
| 0–2 | Отчёт не представлен |

---

## 4. Штрафные санкции

| Нарушение                                                 | Штраф                            |
| --------------------------------------------------------- | -------------------------------- |
| Код не компилируется                                      | −20                              |
| Приложение запускается, но вылетает при добавлении задачи | −10                              |
| Не подписаны скриншоты                                    | −3                               |
| Нарушено именование (классы, файлы, namespace)            | −2 за каждый случай, не более −6 |


---

## 5. Чек-лист преподавателя (быстрая проверка)

- [ ] Проект запускается без ошибок
- [ ] Есть заголовок «Task Manager»
- [ ] Поле ввода и кнопка добавляют задачу
- [ ] Пустая строка не добавляется
- [ ] Отображаются 3–5 тестовых задач
- [ ] Выполненная задача визуально отличается
- [ ] Флажок меняет вид задачи сразу
- [ ] Работает переключатель «Скрыть выполненные»
- [ ] Список прокручивается при большом количестве задач
- [ ] Длинный текст переносится на маленьком экране
- [ ] (Доп.) Есть дата создания
- [ ] (Доп.) Есть приоритет и цветовая индикация
- [ ] Отчёт полный, вывод сформулирован

---

## 6. Вопросы для защиты

Неверные ответы снижают итоговую оценку на один уровень.

1. Почему список задач должен прокручиваться самим `CollectionView`, а не через `ScrollView`?
2. Что произойдёт, если убрать `INotifyPropertyChanged` из модели?
3. Как изменить цвет полосы приоритета для нового значения приоритета?
4. Чем отличаются `Auto` и `*` в описании строк `Grid`?
5. Как реализовать проверку пустой строки при добавлении задачи?
