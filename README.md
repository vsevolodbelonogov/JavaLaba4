## Лабораторная работа №4 — Дженерики, функциональные интерфейсы и MapReduce
Описание проекта

Проект предназначен для изучения работы с дженериками, функциональными интерфейсами и концепцией MapReduce в Java.
Все задания реализованы в отдельных классах, объединённых в один проект.
Входные данные вводятся с клавиатуры с валидацией, демонстрация проводится в Main.java.

Структура проекта

Main.java — демонстрация всех блоков.

Box.java — реализация обобщённого контейнера.

GenericUtils.java — класс с методами filter, map, reduce.

Mapper.java, Reducer.java, Tester.java — функциональные интерфейсы.

InputUtils.java — безопасный ввод данных.

PutPointUtils.java — форматированный вывод результатов.

### Блок 1 — Обобщённая коробка (Box)
Назначение класса

Реализует контейнер, способный хранить один объект произвольного типа.
Позволяет безопасно помещать и извлекать данные с контролем состояния.

Реализация класса
public class Box<T> {
    private T item;

    // Конструктор по умолчанию
    public Box() {}

    /**
     * Помещает объект в коробку.
     * Если коробка уже содержит объект, выбрасывается исключение.
     *
     * @param item объект для помещения
     * @throws IllegalStateException если коробка не пуста
     */
    public void put(T item) {
        if (this.item != null) {
            throw new IllegalStateException("Ошибка: коробка уже содержит объект!");
        }
        this.item = item;
    }

    /**
     * Извлекает объект из коробки и очищает её.
     *
     * @return извлечённый объект
     * @throws IllegalStateException если коробка пуста
     */
    public T get() {
        if (this.item == null) {
            throw new IllegalStateException("Ошибка: коробка пуста!");
        }
        T temp = item;
        item = null;
        return temp;
    }

    /**
     * Проверяет, пуста ли коробка.
     *
     * @return true — если пуста, иначе false
     */
    public boolean isEmpty() {
        return item == null;
    }

    /**
     * Возвращает строковое представление состояния коробки.
     */
    @Override
    public String toString() {
        return isEmpty() ? "[Пустая коробка]" : "[Коробка: " + item + "]";
    }
}

### Блок 2 — Фильтрация данных (GenericUtils и Tester)
Назначение блока

Реализует функционал фильтрации данных произвольного типа на основе пользовательского условия (предиката).

Интерфейс Tester
@FunctionalInterface
public interface Tester<T> {
    /**
     * Проверяет, удовлетворяет ли объект условию.
     *
     * @param value значение для проверки
     * @return true — если условие выполняется, иначе false
     */
    boolean test(T value);
}

Метод filter в GenericUtils
public class GenericUtils {

    /**
     * Фильтрует список по заданному условию.
     * Возвращает новый список, содержащий только элементы,
     * удовлетворяющие тестеру.
     *
     * @param list   исходный список
     * @param tester условие фильтрации
     * @param <T>    тип элементов списка
     * @return новый список, содержащий прошедшие фильтр элементы
     */
    public static <T> List<T> filter(List<T> list, Tester<T> tester) {
        List<T> result = new ArrayList<>();
        for (T item : list) {
            if (tester.test(item)) {
                result.add(item);
            }
        }
        return result;
    }
}

### Блок 3 — MapReduce (Mapper и Reducer)
Назначение блока

Предоставляет реализацию базовых операций map (отображение) и reduce (свёртка) для работы с обобщёнными коллекциями.

Интерфейс Mapper
@FunctionalInterface
public interface Mapper<T, R> {
    /**
     * Преобразует объект входного типа в объект выходного типа.
     *
     * @param input входное значение
     * @return преобразованное значение
     */
    R map(T input);
}

Интерфейс Reducer
@FunctionalInterface
public interface Reducer<T> {
    /**
     * Объединяет два элемента в один в соответствии с заданной логикой.
     *
     * @param a первый элемент
     * @param b второй элемент
     * @return результат объединения
     */
    T reduce(T a, T b);
}

Методы map и reduce в GenericUtils
public class GenericUtils {

    /**
     * Преобразует все элементы списка, применяя указанное отображение.
     *
     * @param list   исходный список
     * @param mapper функция преобразования
     * @param <T>    тип входных данных
     * @param <R>    тип выходных данных
     * @return новый список преобразованных элементов
     */
    public static <T, R> List<R> map(List<T> list, Mapper<T, R> mapper) {
        List<R> result = new ArrayList<>();
        for (T item : list) {
            result.add(mapper.map(item));
        }
        return result;
    }

    /**
     * Последовательно свёртывает список значений в одно, используя заданный редуктор.
     *
     * @param list    список элементов
     * @param reducer логика объединения
     * @param <T>     тип элементов списка
     * @return результат свёртки или null, если список пуст
     */
    public static <T> T reduce(List<T> list, Reducer<T> reducer) {
        if (list.isEmpty()) return null;
        T result = list.get(0);
        for (int i = 1; i < list.size(); i++) {
            result = reducer.reduce(result, list.get(i));
        }
        return result;
    }
}

### Блок 4 — Ввод и вывод (InputUtils и PutPointUtils)
Назначение блока

Обеспечивает надёжный ввод данных с консоли и форматированный вывод информации пользователю.

Класс InputUtils
public class InputUtils {
    private static final Scanner scanner = new Scanner(System.in);

    /**
     * Безопасно считывает целое число с клавиатуры.
     * Повторяет ввод до тех пор, пока пользователь не введёт корректное значение.
     *
     * @param message текст приглашения к вводу
     * @return введённое целое число
     */
    public static int readInt(String message) {
        while (true) {
            System.out.print(message);
            try {
                return Integer.parseInt(scanner.nextLine().trim());
            } catch (NumberFormatException e) {
                System.out.println("Ошибка: введите целое число!");
            }
        }
    }

    /**
     * Считывает строку с клавиатуры.
     *
     * @param message текст приглашения к вводу
     * @return введённая строка
     */
    public static String readLine(String message) {
        System.out.print(message);
        return scanner.nextLine();
    }
}

Класс PutPointUtils
public class PutPointUtils {
    /**
     * Выводит список элементов в читаемом виде.
     *
     * @param title заголовок списка
     * @param list  список элементов
     * @param <T>   тип элементов списка
     */
    public static <T> void printList(String title, List<T> list) {
        System.out.println(title + ":");
        for (T item : list) {
            System.out.println("  → " + item);
        }
    }

    /**
     * Печатает сообщение в консоль с разделителем.
     *
     * @param message текст сообщения
     */
    public static void printBlockTitle(String message) {
        System.out.println("\n--- " + message + " ---");
    }
}

## Итоги

Реализовано:

работа с обобщёнными типами (Generics);

пользовательские функциональные интерфейсы;

реализация map, reduce, filter;

безопасный ввод и форматированный вывод;

принципы инкапсуляции, разделения ответственности и код-стайл Google.

## Автор

ФИО: Белоногов Всеволод Алексеевич
Группа: ИТ-9
Год: 2025
