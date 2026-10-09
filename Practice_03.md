# Практическая работа №3. Стили и ресурсы

| | |
|---|---|
| **Тема** | Создание единого визуального стиля приложения |
| **Среда разработки** | JetBrains Rider |
| **Платформа / язык** | .NET MAUI / C# и XAML |
| **Проект** | Task Manager (продолжение практической работы №2) |
| **Время выполнения** | 2 академических часа (базовая часть) + дополнительное задание |

**Что предполагается.** Практическая работа №2 выполнена, включая дополнительное задание (`TaskPriority`, `CreatedAt`, `PriorityColor`, `PriorityText`). Проект `TaskManager` открывается и запускается. Если дополнительное задание не делали, удалите из шаблона карточки полоску `BoxView` и строку с приоритетом.


---

## 1. Цель работы

Научиться выносить цвета, размеры и оформление элементов в ресурсы и стили, чтобы вся программа выглядела единообразно, а изменение дизайна делалось в одном месте.

## 2. Задачи

1. Создать ресурсы: основной цвет, дополнительный цвет, цвет текста, размеры шрифтов.
2. Создать стили `TitleLabelStyle`, `SecondaryLabelStyle`, `PrimaryButtonStyle`, `TaskCardStyle`.
3. Применить стили к элементам главного экрана.
4. Оформить карточку задачи (отступы, скругление, выделение).
5. Проверить, что общий стиль применён ко всем элементам.
6. (Дополнительно) Реализовать светлую и тёмную темы.

## 3. Краткая теория

| Понятие | Суть |
|---|---|
| **Ресурс** | Именованный объект (цвет, число, отступ, стиль), который хранится в `ResourceDictionary` и берётся по ключу `x:Key`. |
| **Область видимости** | Ресурсы страницы (`ContentPage.Resources`) видны только на ней. Ресурсы приложения (`App.xaml`) видны на всех страницах. |
| **`StaticResource`** | Значение берётся один раз при загрузке XAML. Ресурс должен быть объявлен выше по порядку загрузки. |
| **`DynamicResource`** | Значение отслеживается во время работы и обновляется при замене ресурса. |
| **Style** | Набор `Setter`, применяемый к элементу определённого типа (`TargetType`). |
| **Явный стиль** | Имеет `x:Key`, применяется атрибутом `Style="{StaticResource Key}"`. |
| **Неявный стиль** | Без `x:Key`, применяется автоматически ко всем элементам типа. |
| **`BasedOn`** | Наследование: новый стиль берёт настройки базового и дополняет их. |
| **`AppThemeBinding`** | Выбирает значение по текущей теме (светлая или тёмная). Нужен для дополнительного задания. |

> В шаблоне MAUI уже есть `Resources/Styles/Colors.xaml` и `Styles.xaml` с неявными стилями. Их не удаляем: свои файлы создаём рядом с префиксом `App`, чтобы ключи не совпадали с ключами шаблона.

---

## 4. Ход работы

### Подготовка

1. Откройте решение `TaskManager` в Rider.
2. Создайте отдельную ветку для работы: меню **Git → New Branch…**, имя `lab3-styles`, нажмите **Create**.
3. Запустите приложение и убедитесь, что оно работает так же, как после практической работы №2.

---

### Шаг 1. Создание ресурсов

#### 1.1. Файл цветов

1. В **Solution Explorer** щёлкните правой кнопкой по папке `Resources/Styles` → **Add → New from template…** (или **Add → File**).
2. Введите имя `AppColors.xaml`.
3. Замените содержимое файла:

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<ResourceDictionary xmlns="http://schemas.microsoft.com/dotnet/2021/maui"
                    xmlns:x="http://schemas.microsoft.com/winfx/2009/xaml">

    <!-- Основной цвет бренда -->
    <Color x:Key="AppPrimaryColor">#3949AB</Color>

    <!-- Дополнительный (акцентный) цвет -->
    <Color x:Key="AppSecondaryColor">#F9A826</Color>

    <!-- Цвета текста -->
    <Color x:Key="AppTextColor">#1F2937</Color>
    <Color x:Key="AppMutedTextColor">#5B6472</Color>
    <Color x:Key="AppOnPrimaryColor">#FFFFFF</Color>

    <!-- Фон страницы, карточки и границы -->
    <Color x:Key="AppBackgroundColor">#F4F6FB</Color>
    <Color x:Key="AppCardColor">#FFFFFF</Color>
    <Color x:Key="AppBorderColor">#D5DBE8</Color>

</ResourceDictionary>
```

#### 1.2. Файл размеров и отступов

Создайте файл `Resources/Styles/AppSizes.xaml`:

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<ResourceDictionary xmlns="http://schemas.microsoft.com/dotnet/2021/maui"
                    xmlns:x="http://schemas.microsoft.com/winfx/2009/xaml">

    <!-- Размеры шрифтов -->
    <x:Double x:Key="TitleFontSize">28</x:Double>
    <x:Double x:Key="BodyFontSize">16</x:Double>
    <x:Double x:Key="SecondaryFontSize">13</x:Double>

    <!-- Отступы -->
    <Thickness x:Key="PagePadding">16</Thickness>
    <Thickness x:Key="CardPadding">12</Thickness>
    <Thickness x:Key="CardMargin">0,6</Thickness>

</ResourceDictionary>
```

#### 1.3. Подключение в App.xaml

Откройте `App.xaml` и **добавьте три строки** в список `MergedDictionaries` после шаблонных:

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<Application xmlns="http://schemas.microsoft.com/dotnet/2021/maui"
             xmlns:x="http://schemas.microsoft.com/winfx/2009/xaml"
             xmlns:local="clr-namespace:TaskManager"
             x:Class="TaskManager.App">
    <Application.Resources>
        <ResourceDictionary>
            <ResourceDictionary.MergedDictionaries>
                <ResourceDictionary Source="Resources/Styles/Colors.xaml" />
                <ResourceDictionary Source="Resources/Styles/Styles.xaml" />
                <ResourceDictionary Source="Resources/Styles/AppColors.xaml" />
                <ResourceDictionary Source="Resources/Styles/AppSizes.xaml" />
                <ResourceDictionary Source="Resources/Styles/AppStyles.xaml" />
            </ResourceDictionary.MergedDictionaries>
        </ResourceDictionary>
    </Application.Resources>
</Application>
```

> **Важно.** Порядок имеет значение: `AppStyles.xaml` использует ключи из `AppColors.xaml` и `AppSizes.xaml` через `StaticResource`, поэтому он подключается **последним**. Файл `AppStyles.xaml` создаётся на следующем шаге, а до этого проект не соберётся.

---

### Шаг 2. Создание стилей

Создайте файл `Resources/Styles/AppStyles.xaml`. Обязательные стили из задания: `TitleLabelStyle`, `SecondaryLabelStyle`, `PrimaryButtonStyle`, `TaskCardStyle`. Остальные нужны, чтобы оформить все элементы экрана (шаг 5).

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<ResourceDictionary xmlns="http://schemas.microsoft.com/dotnet/2021/maui"
                    xmlns:x="http://schemas.microsoft.com/winfx/2009/xaml">

    <!-- ===== Страница ===== -->
    <Style x:Key="PageStyle" TargetType="ContentPage">
        <Setter Property="BackgroundColor" Value="{StaticResource AppBackgroundColor}" />
    </Style>

    <!-- ===== Текст ===== -->
    <Style x:Key="TitleLabelStyle" TargetType="Label">
        <Setter Property="TextColor" Value="{StaticResource AppPrimaryColor}" />
        <Setter Property="FontSize" Value="{StaticResource TitleFontSize}" />
        <Setter Property="FontAttributes" Value="Bold" />
    </Style>

    <Style x:Key="BodyLabelStyle" TargetType="Label">
        <Setter Property="TextColor" Value="{StaticResource AppTextColor}" />
        <Setter Property="FontSize" Value="{StaticResource BodyFontSize}" />
        <Setter Property="LineBreakMode" Value="WordWrap" />
    </Style>

    <Style x:Key="SecondaryLabelStyle" TargetType="Label">
        <Setter Property="TextColor" Value="{StaticResource AppMutedTextColor}" />
        <Setter Property="FontSize" Value="{StaticResource SecondaryFontSize}" />
    </Style>

    <!-- Наследование: название задачи = обычный текст + жирное начертание -->
    <Style x:Key="TaskTitleLabelStyle" TargetType="Label"
           BasedOn="{StaticResource BodyLabelStyle}">
        <Setter Property="FontAttributes" Value="Bold" />
    </Style>

    <!-- ===== Кнопка ===== -->
    <Style x:Key="PrimaryButtonStyle" TargetType="Button">
        <Setter Property="BackgroundColor" Value="{StaticResource AppPrimaryColor}" />
        <Setter Property="TextColor" Value="{StaticResource AppOnPrimaryColor}" />
        <Setter Property="FontSize" Value="{StaticResource BodyFontSize}" />
        <Setter Property="FontAttributes" Value="Bold" />
        <Setter Property="CornerRadius" Value="10" />
        <Setter Property="Padding" Value="16,8" />
        <Setter Property="MinimumHeightRequest" Value="44" />
        <Setter Property="VisualStateManager.VisualStateGroups">
            <VisualStateGroupList>
                <VisualStateGroup x:Name="CommonStates">
                    <VisualState x:Name="Normal" />
                    <VisualState x:Name="Pressed">
                        <VisualState.Setters>
                            <Setter Property="Opacity" Value="0.8" />
                        </VisualState.Setters>
                    </VisualState>
                    <VisualState x:Name="Disabled">
                        <VisualState.Setters>
                            <Setter Property="BackgroundColor" Value="{StaticResource AppBorderColor}" />
                        </VisualState.Setters>
                    </VisualState>
                </VisualStateGroup>
            </VisualStateGroupList>
        </Setter>
    </Style>

    <!-- ===== Элементы ввода ===== -->
    <Style x:Key="EntryStyle" TargetType="Entry">
        <Setter Property="TextColor" Value="{StaticResource AppTextColor}" />
        <Setter Property="PlaceholderColor" Value="{StaticResource AppMutedTextColor}" />
        <Setter Property="FontSize" Value="{StaticResource BodyFontSize}" />
    </Style>

    <Style x:Key="PickerStyle" TargetType="Picker">
        <Setter Property="TextColor" Value="{StaticResource AppTextColor}" />
        <Setter Property="TitleColor" Value="{StaticResource AppMutedTextColor}" />
        <Setter Property="FontSize" Value="{StaticResource BodyFontSize}" />
    </Style>

    <Style x:Key="SwitchStyle" TargetType="Switch">
        <Setter Property="OnColor" Value="{StaticResource AppSecondaryColor}" />
    </Style>

    <Style x:Key="TaskCheckBoxStyle" TargetType="CheckBox">
        <Setter Property="Color" Value="{StaticResource AppPrimaryColor}" />
    </Style>

    <!-- ===== Карточка задачи ===== -->
    <Style x:Key="TaskCardStyle" TargetType="Border">
        <Setter Property="BackgroundColor" Value="{StaticResource AppCardColor}" />
        <Setter Property="Stroke" Value="{StaticResource AppBorderColor}" />
        <Setter Property="StrokeThickness" Value="1" />
        <Setter Property="StrokeShape" Value="RoundRectangle 12" />
        <Setter Property="Padding" Value="{StaticResource CardPadding}" />
        <Setter Property="Margin" Value="{StaticResource CardMargin}" />
    </Style>

</ResourceDictionary>
```

**Что важно понимать**

- Каждый `Setter` задаёт значение одного свойства. Ссылка `{StaticResource ...}` берёт значение из ресурсов.
- `TaskTitleLabelStyle` не повторяет настройки: `BasedOn` берёт цвет, размер и перенос строк из `BodyLabelStyle`.
- `VisualStateManager` внутри `PrimaryButtonStyle` задаёт вид кнопки при нажатии (`Pressed`) и в неактивном состоянии (`Disabled`).
- `MinimumHeightRequest="44"` делает кнопку удобной для нажатия пальцем.

Теперь соберите проект (**Build → Build Solution**). Он должен собраться без ошибок: стили объявлены, но пока нигде не применены.


---

### Шаг 3. Применение стилей

Замените содержимое `MainPage.xaml` целиком. Код-behind (`MainPage.xaml.cs`) **не меняется**.

```xml
<?xml version="1.0" encoding="utf-8" ?>
<ContentPage xmlns="http://schemas.microsoft.com/dotnet/2021/maui"
             xmlns:x="http://schemas.microsoft.com/winfx/2009/xaml"
             xmlns:models="clr-namespace:TaskManager.Models"
             x:Class="TaskManager.MainPage"
             Style="{StaticResource PageStyle}"
             Shell.NavBarIsVisible="False">

    <Grid RowDefinitions="Auto,Auto,*"
          RowSpacing="12"
          Padding="{StaticResource PagePadding}">

        <!-- Строка 0: заголовок -->
        <VerticalStackLayout Grid.Row="0" Spacing="2">
            <Label Text="Task Manager"
                   Style="{StaticResource TitleLabelStyle}" />
            <Label Text="Список задач"
                   Style="{StaticResource SecondaryLabelStyle}" />
        </VerticalStackLayout>

        <!-- Строка 1: ввод, кнопка, приоритет, переключатель -->
        <VerticalStackLayout Grid.Row="1" Spacing="8">

            <Grid ColumnDefinitions="*,Auto" ColumnSpacing="8">
                <Entry x:Name="TaskEntry"
                       Grid.Column="0"
                       Placeholder="Введите задачу"
                       ReturnType="Done"
                       Style="{StaticResource EntryStyle}"
                       Completed="OnAddClicked" />
                <Button Grid.Column="1"
                        Text="Добавить"
                        Style="{StaticResource PrimaryButtonStyle}"
                        Clicked="OnAddClicked" />
            </Grid>

            <Picker x:Name="PriorityPicker"
                    Title="Приоритет"
                    Style="{StaticResource PickerStyle}" />

            <HorizontalStackLayout Spacing="8">
                <Switch x:Name="HideCompletedSwitch"
                        Style="{StaticResource SwitchStyle}"
                        Toggled="OnHideCompletedToggled" />
                <Label Text="Скрыть выполненные"
                       Style="{StaticResource BodyLabelStyle}"
                       VerticalOptions="Center" />
            </HorizontalStackLayout>

        </VerticalStackLayout>

        <!-- Строка 2: список задач (область прокрутки) -->
        <CollectionView x:Name="TasksView"
                        Grid.Row="2"
                        SelectionMode="None">
            <CollectionView.ItemTemplate>
                <!-- Шаблон карточки добавляется на шаге 4 -->
                <DataTemplate x:DataType="models:TaskItem">
                    <Label Text="{Binding Title}"
                           Style="{StaticResource BodyLabelStyle}" />
                </DataTemplate>
            </CollectionView.ItemTemplate>
        </CollectionView>

    </Grid>
</ContentPage>
```

Запустите приложение. Заголовок, кнопка, поле ввода и переключатель должны получить единое оформление.


---

### Шаг 4. Оформление карточки задачи

Замените `DataTemplate` внутри `CollectionView` на карточку. Она использует `TaskCardStyle` (внутренние и внешние отступы, скругление, рамка), а выделение даёт цветная полоса приоритета и лёгкая тень.

```xml
<CollectionView.ItemTemplate>
    <DataTemplate x:DataType="models:TaskItem">

        <Border Style="{StaticResource TaskCardStyle}">

            <!-- Визуальное выделение: тень -->
            <Border.Shadow>
                <Shadow Brush="Black"
                        Opacity="0.12"
                        Radius="8"
                        Offset="0,2" />
            </Border.Shadow>

            <!-- Выполненная задача становится бледнее -->
            <Border.Triggers>
                <DataTrigger TargetType="Border"
                             Binding="{Binding IsCompleted}"
                             Value="True">
                    <Setter Property="Opacity" Value="0.6" />
                </DataTrigger>
            </Border.Triggers>

            <Grid ColumnDefinitions="6,Auto,*" ColumnSpacing="10">

                <!-- Цветовая индикация приоритета -->
                <BoxView Grid.Column="0"
                         Color="{Binding PriorityColor}"
                         CornerRadius="3" />

                <CheckBox Grid.Column="1"
                          IsChecked="{Binding IsCompleted}"
                          Style="{StaticResource TaskCheckBoxStyle}"
                          VerticalOptions="Center" />

                <VerticalStackLayout Grid.Column="2" Spacing="2">

                    <Label Text="{Binding Title}"
                           Style="{StaticResource TaskTitleLabelStyle}">
                        <Label.Triggers>
                            <DataTrigger TargetType="Label"
                                         Binding="{Binding IsCompleted}"
                                         Value="True">
                                <Setter Property="TextDecorations" Value="Strikethrough" />
                                <Setter Property="TextColor"
                                        Value="{StaticResource AppMutedTextColor}" />
                            </DataTrigger>
                        </Label.Triggers>
                    </Label>

                    <HorizontalStackLayout Spacing="8">
                        <Label Text="{Binding PriorityText, StringFormat='Приоритет: {0}'}"
                               Style="{StaticResource SecondaryLabelStyle}"
                               TextColor="{Binding PriorityColor}" />
                        <Label Text="{Binding CreatedAt, StringFormat='{0:dd.MM.yyyy HH:mm}'}"
                               Style="{StaticResource SecondaryLabelStyle}" />
                    </HorizontalStackLayout>

                </VerticalStackLayout>
            </Grid>
        </Border>

    </DataTemplate>
</CollectionView.ItemTemplate>
```

**Как карточка выполняет требования задания**

| Требование | Где реализовано |
|---|---|
| Внутренние отступы | `Padding` в `TaskCardStyle` (ресурс `CardPadding`) |
| Внешний отступ | `Margin` в `TaskCardStyle` (ресурс `CardMargin`) |
| Скругление | `StrokeShape="RoundRectangle 12"` |
| Визуальное выделение | Рамка `Stroke`, тень `Border.Shadow`, цветная полоса приоритета, полупрозрачность выполненной задачи |

Запустите приложение и проверьте: карточки выровнены, у них есть отступы и скруглённые углы, выполненная задача зачёркнута и бледнее остальных.

![Рис. 6 — Список задач с оформленными карточками](images/fig6.png)

**Ориентир для сравнения: палитры светлой и тёмной тем**

| Элемент | Светлая тема | Тёмная тема |
|---|---|---|
| Основной цвет (заголовок, кнопка, флажки) | `#3949AB` | `#8C9EFF` |
| Дополнительный цвет (переключатель) | `#F9A826` | `#F9A826` |
| Текст | `#1F2937` | `#F1F5F9` |
| Приглушённый текст | `#5B6472` | `#A8B3C7` |
| Текст на основной кнопке | `#FFFFFF` | `#10162B` |
| Фон страницы | `#F4F6FB` | `#121826` |
| Фон карточки | `#FFFFFF` | `#1E2638` |
| Граница карточки | `#D5DBE8` | `#34405A` |

---

### Шаг 5. Проверка всех страниц и применение общего стиля

Цель шага: убедиться, что в разметке не осталось «зашитых» цветов и размеров, а все элементы берут оформление из ресурсов.

#### 5.1. Поиск «лишних» значений

1. В Rider нажмите **Ctrl+Shift+F** (Find in Files).
2. Включите **File mask:** `*.xaml` и **Directory:** весь проект.
3. Поочерёдно ищите:

| Поисковая строка | Что делать с найденным |
|---|---|
| `TextColor="` | Заменить стилем (`Style=`) или оставить, если цвет приходит из привязки (`{Binding ...}`) |
| `BackgroundColor="` | Заменить стилем или ресурсом |
| `FontSize="` | Заменить стилем или ресурсом (`TitleFontSize`, `BodyFontSize`, `SecondaryFontSize`) |
| `Color="#` и `Stroke="` | Заменить ресурсом (`AppBorderColor` и т. п.) |
| `Padding="` и `Margin="` | Если значение повторяется, вынести в ресурс `Thickness` |

Исключения (допустимо оставить): значения из `Binding` (цвет приоритета) остаются в разметке. `Padding` корневого `Grid` на странице уже взят из ресурса `PagePadding`.

#### 5.2. Проверка по спискам страниц

| Файл | Что проверить | Статус |
|---|---|---|
| `MainPage.xaml` | Все `Label`, `Entry`, `Picker`, `Switch`, `CheckBox`, `Button`, `Border` имеют `Style` | |
| `AppShell.xaml` | Нет собственных цветов, заголовок скрыт (`Shell.NavBarIsVisible="False"`) | |
| Другие страницы (если есть) | Установлен `Style="{StaticResource PageStyle}"` и стили элементов | |

#### 5.3. Проверка на экране небольшого размера

Запустите приложение на эмуляторе с маленьким экраном (Pixel 2 или аналогичным) и убедитесь, что:

- карточки не выходят за границы экрана;
- длинное название задачи переносится на следующую строку;
- кнопка и поле ввода не перекрываются;
- список прокручивается при 15–20 задачах.

#### 5.4. Эксперимент: изменение дизайна в одном месте

Это главное доказательство пользы ресурсов:

1. Откройте `AppColors.xaml` и измените `AppPrimaryColor` на `#00796B`.
2. Запустите приложение: заголовок, кнопка и флажки поменяли цвет, хотя `MainPage.xaml` не трогали.
3. Верните исходное значение.


---

## 5. Дополнительное задание. Светлая и тёмная темы

### 5.1. Цвета для тёмной темы

Добавьте в `AppColors.xaml` (перед закрывающим тегом `</ResourceDictionary>`) тёмные аналоги:

```xml
    <!-- ===== Тёмная тема ===== -->
    <Color x:Key="AppPrimaryColorDark">#8C9EFF</Color>
    <Color x:Key="AppTextColorDark">#F1F5F9</Color>
    <Color x:Key="AppMutedTextColorDark">#A8B3C7</Color>
    <Color x:Key="AppOnPrimaryColorDark">#10162B</Color>
    <Color x:Key="AppBackgroundColorDark">#121826</Color>
    <Color x:Key="AppCardColorDark">#1E2638</Color>
    <Color x:Key="AppBorderColorDark">#34405A</Color>
```

Дополнительный цвет `AppSecondaryColor` (янтарный) одинаково хорошо виден на обоих фонах, поэтому для него тёмный вариант не нужен.

### 5.2. Замена значений в стилях через AppThemeBinding

В `AppStyles.xaml` каждое значение, зависящее от темы, должно принять такой вид:

```xml
<!-- было -->
<Setter Property="TextColor" Value="{StaticResource AppTextColor}" />

<!-- стало -->
<Setter Property="TextColor"
        Value="{AppThemeBinding Light={StaticResource AppTextColor},
                                Dark={StaticResource AppTextColorDark}}" />
```

Править все места вручную долго и легко ошибиться. Используйте **замену по регулярному выражению** в Rider:

1. Откройте `AppStyles.xaml`, нажмите **Ctrl+R** (Replace in File).
2. Включите режим **Regex** (иконка `.*`).
3. В поле поиска введите:

```text
\{StaticResource (App(?:Primary|Text|MutedText|OnPrimary|Background|Card|Border)Color)\}
```

4. В поле замены введите:

```text
{AppThemeBinding Light={StaticResource $1}, Dark={StaticResource $1Dark}}
```

5. Нажмите **Replace All**.

> **Что делает регулярное выражение.** Оно находит ссылки на ресурсы, у которых есть тёмный вариант, и для каждой подставляет пару «светлый / тёмный». Группа `$1` содержит найденное имя (например, `AppTextColor`), а суффикс `Dark` даёт имя тёмного аналога.

6. Проверьте результат: ссылки на `AppSecondaryColor` и на размеры шрифтов остались без изменений.

Фрагмент `TaskCardStyle` после замены должен выглядеть так:

```xml
<Style x:Key="TaskCardStyle" TargetType="Border">
    <Setter Property="BackgroundColor"
            Value="{AppThemeBinding Light={StaticResource AppCardColor}, Dark={StaticResource AppCardColorDark}}" />
    <Setter Property="Stroke"
            Value="{AppThemeBinding Light={StaticResource AppBorderColor}, Dark={StaticResource AppBorderColorDark}}" />
    <Setter Property="StrokeThickness" Value="1" />
    <Setter Property="StrokeShape" Value="RoundRectangle 12" />
    <Setter Property="Padding" Value="{StaticResource CardPadding}" />
    <Setter Property="Margin" Value="{StaticResource CardMargin}" />
</Style>
```

В `MainPage.xaml` есть одно место, которое нужно исправить вручную: триггер выполненной задачи.

```xml
<Setter Property="TextColor"
        Value="{AppThemeBinding Light={StaticResource AppMutedTextColor},
                                Dark={StaticResource AppMutedTextColorDark}}" />
```

### 5.3. Переключатель темы на странице

В `MainPage.xaml` замените блок заголовка (строка 0 основной сетки) на сетку из двух колонок:

```xml
<!-- Строка 0: заголовок и переключатель темы -->
<Grid Grid.Row="0" ColumnDefinitions="*,Auto">

    <VerticalStackLayout Grid.Column="0" Spacing="2">
        <Label Text="Task Manager"
               Style="{StaticResource TitleLabelStyle}" />
        <Label Text="Список задач"
               Style="{StaticResource SecondaryLabelStyle}" />
    </VerticalStackLayout>

    <HorizontalStackLayout Grid.Column="1" Spacing="6" VerticalOptions="Start">
        <Label Text="Тёмная"
               Style="{StaticResource SecondaryLabelStyle}"
               VerticalOptions="Center" />
        <Switch x:Name="DarkThemeSwitch"
                Style="{StaticResource SwitchStyle}"
                Toggled="OnDarkThemeToggled" />
    </HorizontalStackLayout>

</Grid>
```

### 5.4. Код-behind

В `MainPage.xaml.cs` добавьте в конструктор установку начального положения переключателя и новый обработчик:

```csharp
public MainPage()
{
    InitializeComponent();

    PriorityPicker.ItemsSource = new List<string> { "Низкий", "Средний", "Высокий" };
    PriorityPicker.SelectedIndex = 1;

    // Переключатель отражает текущую тему системы
    DarkThemeSwitch.IsToggled = Application.Current?.RequestedTheme == AppTheme.Dark;

    LoadTestData();
    RefreshList();
}

// Переключение темы вручную
private void OnDarkThemeToggled(object? sender, ToggledEventArgs e)
{
    if (Application.Current is null)
        return;

    Application.Current.UserAppTheme = e.Value ? AppTheme.Dark : AppTheme.Light;
}
```

Свойство `UserAppTheme` переопределяет системную тему. Значение `Unspecified` возвращает приложение к настройке устройства.

### 5.5. Проверка

1. Запустите приложение и включите переключатель **Тёмная**: фон, карточки, текст и кнопка меняют цвета.
2. Переключите тему в самом эмуляторе (**Settings → Display → Dark theme**), не касаясь переключателя в приложении: оформление должно следовать системе.
3. Убедитесь, что везде читаются подписи, а цветная полоса приоритета видна на обоих фонах.

### 5.6. Запоминание выбора темы (по желанию)

В `OnDarkThemeToggled` сохраняйте выбор:

```csharp
Preferences.Default.Set("dark_theme", e.Value);
```

А в конструкторе `App` (файл `App.xaml.cs`) после `InitializeComponent()` восстанавливайте:

```csharp
UserAppTheme = Preferences.Default.Get("dark_theme", false) ? AppTheme.Dark : AppTheme.Light;
```

---

## 6. Типичные ошибки

| Симптом | Причина и решение |
|---|---|
| `StaticResource not found for key AppPrimaryColor` | Неверный порядок в `App.xaml`: `AppColors.xaml` и `AppSizes.xaml` должны идти **перед** `AppStyles.xaml` |
| `XamlParseException` при запуске | Опечатка в ключе (`TitleLabelStyle` и `TitleLabeStyle`) или ссылка на стиль, который ещё не создан |
| Стиль «не применяется» | У элемента задан локальный атрибут (`TextColor="..."`), который перекрывает стиль. Удалите атрибут |
| Стиль для `Label` не подходит к `Button` | Стиль работает только для типа из `TargetType`. Нужны отдельные стили |
| Ошибка в `x:Double` или `Thickness` | Отсутствует `xmlns:x` в начале файла ресурсов |
| Тёмная тема не меняет цвета | Остались `StaticResource` вместо `AppThemeBinding`, либо в `MainPage.xaml` остался фиксированный цвет |
| Новый файл не виден компилятору | Проверьте, что действие сборки (**Build Action**) файла `MauiXaml` (в Rider: правый клик по файлу → **Properties**) |
| Не работает регулярное выражение | Включён режим Regex? Выделен именно файл `AppStyles.xaml`? Нет лишних пробелов в поле поиска? |

## 7. Содержание отчёта

1. Титульный лист, тема, цель работы.
2. Скриншоты (рис. 1–12) с подписями.
3. Листинги: `AppColors.xaml`, `AppSizes.xaml`, `AppStyles.xaml`, `App.xaml`, `MainPage.xaml`, `MainPage.xaml.cs`.
4. Таблица проверки шага 5.2.
5. Описание эксперимента из шага 5.4 (что изменили и что произошло).
6. Ответы на контрольные вопросы.
7. Вывод.

## 8. Контрольные вопросы

1. Что такое ресурс в XAML и где он может быть объявлен?
2. Чем `StaticResource` отличается от `DynamicResource`?
3. Чем явный стиль отличается от неявного? Что даёт ключ `x:Key`?
4. Для чего нужен атрибут `BasedOn`? Приведите пример из вашей работы.
5. Почему порядок подключения словарей в `App.xaml` важен?
6. Какие преимущества даёт вынос цветов и размеров в ресурсы?
7. Как работает `AppThemeBinding` и откуда берётся значение темы?
8. В чём разница между `UserAppTheme` и `RequestedTheme`?

## 9. Список источников

1. Документация .NET MAUI — Microsoft Learn: <https://learn.microsoft.com/dotnet/maui/>
2. Разделы «Resource dictionaries», «Styles» и «Theming» документации .NET MAUI.
3. Документация JetBrains Rider — .NET MAUI: <https://www.jetbrains.com/help/rider/>
