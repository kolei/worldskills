# Проект "Калькулятор"

Чтобы не обрабатывать каждую кнопку отдельно, можно при создании первого оутлета выбрать в connection вместо "outlet" "outlet collection" 

![](img/collection.png)

Сделать связь созданной коллекции с нужными кнопками и затем обрабатывать списком

![](img/callback.png)

```swift
for outlet in outletCollection {
    outlet.layer.borderWidth = 1
    outlet.layer.borderColor = UIColor.darkGray.cgColor
}
```

# Проект "погода"

Создаем новый проект

## добавление фона
* картинку перенести в папку Assets.xcassets В ПРИЛОЖЕНИИ
* на контроллер кладем ImageView
* выбираем загруженный фон

## делаем дополнительный контроллер для таблицы
* создаем КНОПКУ для перехода на следующий экран
* создаем контроллер TableView (так же как кнопки)
* можем задать таблице фон ``tableView.backgroundView = UIImageView.init(image: UIImage(named: "background3"))``
* кнопку связываем с контроллером (Show Detail)
* создаем новый swift файл для класса контроллера (File - New - File - Cocoa Touch Class); наследуемся от класса UITavleVewController
* в свойствах ячейки указываем Identifier "cityCell"

```swift
var cityList = [String]()

cityList.append("Москва")
tableView.reloadData()

override func numberOfSections(in tableView: UITableView) -> Int {
// #warning Incomplete implementation, return the number of sections
return 1
}

override func tableView(_ tableView: UITableView, numberOfRowsInSection section: Int) -> Int {
// #warning Incomplete implementation, return the number of rows
return cityList.count
}


override func tableView(_ tableView: UITableView, cellForRowAt indexPath: IndexPath) -> UITableViewCell {
let cell = tableView.dequeueReusableCell(withIdentifier: "cityCell", for: indexPath)

cell.textLabel?.text = cityList[indexPath.row]

return cell
}
```
## кастомизация отображения ячейки
* создаем новый swift файл, наследующий класс UITableViewCell
* в свойствах ячейки указываем созданный класс
* в ячейку добавляем нужные объекты (label...)
* outlet-ы объектов добавляем к классу

## меняем вывод названия города

```swift
override func tableView(_ tableView: UITableView, cellForRowAt indexPath: IndexPath) -> UITableViewCell {
let cell = tableView.dequeueReusableCell(withIdentifier: "cityCell", for: indexPath) as! TableViewCell

cell.cityLabel.text = cityList[indexPath.row]

return cell
}
```

## делаем добавление и удаление городов
* добавляем навбар и навбатттон
* от навбаттона делаем action в контроллер (перетаскиваем с ctrl ниже viewDidLoad)

//добавляем код в созданный action
```swift
@IBAction func addCity(_ sender: Any) {
    //создаем объекет Alert
    let alert = UIAlertController(title: "Добавление город", message: "Добавьте пожалуйста город!", preferredStyle: .alert)
    
    //добавляем к алерту текстовое поле
    alert.addTextField { (textField) in }
    
    //добавляем действие на кнопку ОК
    alert.addAction(UIAlertAction(title: "OK", style: UIAlertAction.Style.default, handler: { (alertAction) in
        //в алерте может быть несколько текстовых полей, поэтому нужно вытаскиваем по индексу
        let textField = alert.textFields![0] as UITextField
        
        //в свой список городов добавляем новый элемент
        self.cityList.append(textField.text!)
        
        //обновляем отображение
        self.tableView.reloadData()
        }))

    //создаем действие на кнопку отмены
    alert.addAction(UIAlertAction(title: "Cancel", style: .cancel, handler: { (alertAction) in }))
    
    self.present(alert, animated: true, completion: nil)
}
```

* переопределяем действие для удаления ячейки

```swift 
override func tableView(_ tableView: UITableView, commit editingStyle:  UITableViewCell.EditingStyle, forRowAt indexPath: IndexPath) {
    if editingStyle == .delete {
        // Delete the row from the data source

        cityList.remove(at: indexPath.row)
        tableView.deleteRows(at: [indexPath], with: .fade)
        tableView.reloadData()


    } else if editingStyle == .insert {
        // Create a new instance of the appropriate class, insert it into the array, and add a new row to the table view
    }
}
```

* переопределяем событие при выборе элемента списка
```swift
override func prepare(for segue: UIStoryboardSegue, sender: Any?) {
    if segue.identifier == "toMain" {
        let vc = segue.destination as! ViewController
        let index = self.tableView.indexPathForSelectedRow
        
        //обратите внимание - в ViewController должна быть объявлена переменная city
        vc.city = cityList[(index?.row)!]
    }
}
```

## Подключаем апи для получения прогноза погоды

### Добавляем библиотеки

1 устанавливаем поддержку cocoapods (если еще не установлен)
``sudo gem install cocoapods``

2 инициализируем pod  для нашего проекта (в каталоге с файлом проекта *.xcodeproj)
``pod init``

3 в созданном файле (Podfile) добавляем зависимости (названия библиотек) для работы с http и json
```
# проверяем название целевого проекта
target 'weather' do

...

# Pods for weather
pod 'Alamofire'
pod 'SwiftyJSON'

...
```

4 Устанавливаем зависимости
 ```pod install```
 
 5 Теперь вместо проекта нужно запускать *.xcworkspace
 
 6 Пересобираем проект, чтобы проверить ошибки версий библиотек (Product - Build)
 
 7 В ViewController.swift добавляем импорт нужных библиотек
```swift
import Alamofire
import SwiftyJSON
```

## Получение температуры и иконки (солнышко, тучки...)
С гитхаба копипастим пример запроса и заворачиваем его в функцию (ViewController)
```swift
func downloadData(city: String) {
    // токен для АПИ
    let token = "1e936ee21707e2a418e98dca00877357"

    // УРЛ с городом, метрической системой и токеном
    let urlStr = "https://api.openweathermap.org/data/2.5/weather?q=\(city)&units=metric&appid=\(token)"

    // перекодируем адрес (URLencode)
    let url = urlStr.addingPercentEncoding(withAllowedCharacters: .urlQueryAllowed)

    // посылаем запрос
    Alamofire.request(url!, method: .get).validate().responseJSON { response in
        switch response.result {
            case .success(let value):
                //если запрос выполнен успешно, то разбираем ответ и вытаскиваем нужные данные
                let json = JSON(value)
                self.temp = json["main"]["temp"].stringValue
                self.lblTemp.text = json["main"]["temp"].stringValue+"°C"
                
                //можно сразу запросить иконку (только нужно хранить название предыдущей, чтобы не запрашивать постоянно)
                let icoName = json["weather"][0]["icon"].stringValue
                
                if let urlStr2 = URL(string: "https://openweathermap.org/img/w/\(icoName).png") {
                    if let data = try? Data(contentsOf: urlStr2){
                        self.icoWeather.image = UIImage(data: data)
                    }
                } 
                
            case .failure(let error):
                print(error)
        }
    }
}
```
 
Запрашиваем температуру при отрисовке экрана
```swift
override func viewDidLoad() {
...
    downloadData(city: city)
```

##Простой способ хранить несекурные данные (последний выбранный город и список городов)

```swift
// получение ранее сохраненных данных
let city = UserDefaults.standard.string(forKey: "lastCity") ?? "Москва"

let savedList = UserDefaults.standard.stringArray(forKey: "cityList") ?? [String]()
for city in savedList {
    cityList.append(city)
}

// при изменении списка (добавили, удалили город) пересохраняем 
UserDefaults.standard.set(self.cityList, forKey: "cityList")

// при выборе города аналогично
UserDefaults.standard.set(cityList[(index?.row)!], forKey: "lastCity")
```

# Проект "tvOS" - погода на телевизоре

При создании проекта выбираем шаблон "tvOS" и так же "Single View App"

![](img/tvos-create.png)

После создания файла для контроллера (второй экран) не забываем в свойствах контроллера указать созданный класс

![](img/setclass.png)

Для поиска введенного города нужно связать текстовое поле с экземпляром класса (``tfCityName.delegate = self``) и реализовать функцию перехватчик ``textFieldShouldReturn``

```swift
override func viewDidLoad() {
    super.viewDidLoad()

    // связываем с собой, чтобы сработала функция "textFieldShouldReturn"
    tfCityName.delegate = self
}

func textFieldShouldReturn(_ textField: UITextField) -> Bool {
    searchCity(city: tfCityName.text!)
    return true
}
```

Чтобы при возврате на первый экран обновить информацию нужно переопределеить функцию ``viewWillAppear`` и снова вызвать функцию поиска погоды

```swift
override func viewWillAppear(_ animated: Bool) {
    loadWeatherInfo()
}
```
# "Погода" на часах

![](img/watch-create.png)

* Добавляем права для получения данных с сервера

![](img/rights.png)

При связи кнопки со вторым экраном выбираем "push"
