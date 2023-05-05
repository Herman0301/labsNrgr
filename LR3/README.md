
**1. Встановлення Terraform**

Цей абзац не потребує особливого пояснення. Terraform можна встановити за допомогою інсталятора, який надає розробник, або через менеджер пакетів. У цьому випадку ми оберемо встановлення terraform вручну. Щоб переконатися у коректності установки, достатньо виконати будь-яку команду. Наприклад,

terraform -version

Тоді отримаємо такий вивід 

![](Aspose.Words.cb0b4cf3-caa9-4d6c-b4f2-c0908b76d88d.001.png)

**2. Автоматизація створення віртуальної машини**

Тепер створимо віртуальну машину настільки автоматизовано, наскільки це є можливим. Для цього треба виконати декілька дій.

**2.1. Створення нового проекту**

Створимо новий проект, щоб попередні налаштування нам не заважали. 

![](Aspose.Words.cb0b4cf3-caa9-4d6c-b4f2-c0908b76d88d.002.png)



**2.2. Створення service account**

Для того, щоб забезпечити можливість виконання terraform певних завдань у нашому новому проекті та забезпечити GCP інформацію про перевірений сервіс, потрібно створити service account. Для цього можна перейти до пункту Dashboard -> Service Account, натиснути кнопку Create Service Account, вказати назву акаунту, призначити йому ідентифікатор та ролі. Як результат, буде створений новий акаунт, зареєстрований у системі.

![](Aspose.Words.cb0b4cf3-caa9-4d6c-b4f2-c0908b76d88d.003.png)

**2.3. Створення ключу доступу**

Далі там треба зайти в Actions -> Manage keys і створити там новий ключ в форматі JSON.

![](Aspose.Words.cb0b4cf3-caa9-4d6c-b4f2-c0908b76d88d.004.png)

Тепер можна перейти до конфігурації terraform.

**2.4. Налаштування Terraform**

Тепер нам потрібно створити робочу директорію для terraform. Створюємо її будь-яким зручним методом.

Далі ми створимо: main.tf,  Спочатку попрацюємо з основним файлом main.tf із основною конфігурацією.

Щоб створити віртуальну машину за необхідними умовами, напишемо наступний код

terraform {

`  `required\_providers {

`    `google = {

`      `source = "hashicorp/google"

`      `version = "4.51.0"

`    `}

`  `}

}

provider "google" {

`  `credentials = file("mygcpkey.json")

`  `project = "minecraft-1"

`  `region  = "us-central1"

`  `zone    = "us-central1-c"

}

resource "google\_compute\_network" "vpc\_network" {

`  `name = "minecraft-vpc "

`  `project = "terform"

}

resource "google\_compute\_subnetwork" "subnet-1" {

`  `name          = "subnet-1"

`  `network       = google\_compute\_network.vpc\_network.id

`  `ip\_cidr\_range = "10.2.0.0/16"

`  `region        = "us-central1"

}

resource "google\_compute\_instance" "my\_server" {

`  `name         = "minecraft "

`  `machine\_type = "f1-micro"

`  `tags = ["khai", "linux", "devops", "ukraine"]


`  `boot\_disk {

`    `initialize\_params {

`      `image = "debian-cloud/debian-11"

`    `}

`  `}

`  `network\_interface {

`    `network = google\_compute\_network.vpc\_network.name

`    `access\_config {

`    `}

`  `}

}

resource "google\_compute\_firewall" "vpc-network-allow" {

`  `name    = "letmein"

`  `network = google\_compute\_network.vpc\_network.self\_link

`  `allow {

`    `protocol = "tcp"

`    `ports    = ["80", "8080", "1000-2000"]

`  `}

`  `target\_tags = ["http-server","https-server"]

`  `source\_tags = ["vpc-network-allow"]

}

Файл outputs.tf виглядають наступним чином.

outputs.tf

output "ip\_intra" {

`  `value = google\_compute\_instance.my\_server.network\_interface.0.network\_ip

}

output "ip\_extra" {

value = google\_compute\_instance.my\_server.network\_interface.0.access\_config.0.nat\_ip

}

Тепер маємо змогу зайти до директорії через термінал та прописати команду

terraform init

Отримуємо наступне повідо	млення, що свідчить про коректну ініціалізацію директорії.

![](Aspose.Words.cb0b4cf3-caa9-4d6c-b4f2-c0908b76d88d.005.png) 

Далі пишемо

terraform apply

І отримуємо список того, що планує зробити terraform.

![](Aspose.Words.cb0b4cf3-caa9-4d6c-b4f2-c0908b76d88d.006.png)

Якщо нас все влаштовує, пишемо "yes" і чекаємо на створення віртуальної машини і її структури.

Все пройшло коректно і від terraform ми отримали таке повідомлення.

Ip\_extra= “104.197.16.29”

Ip\_intra = “10.128.0.2”

Помітимо, що у машини є дві ip-адреси. Одна з них -- внутрішня адреса у VPC мережі Google, а інша -- адреса NAT, тобто можна сказати, що вона глобальна і через неї можна під'єднатися до нашої ВМ ззовні.

Подивимося на те, чи вірно все було створено.

![](Aspose.Words.cb0b4cf3-caa9-4d6c-b4f2-c0908b76d88d.007.png)

` `![](Aspose.Words.cb0b4cf3-caa9-4d6c-b4f2-c0908b76d88d.008.png)

![](Aspose.Words.cb0b4cf3-caa9-4d6c-b4f2-c0908b76d88d.009.png) ![](Aspose.Words.cb0b4cf3-caa9-4d6c-b4f2-c0908b76d88d.010.png)

Ми бачимо, що все було виконано правильно: машина була належним чином створена і отримала всі адреси, які були виведені у консоль. Брандмауер дозволив з'єднання http та https, а також була створена subnetwork з параметрами, які ми встановили. Отже, все заплановане було виконано належним чином. Ми можемо знищити нашу віртуальну машину за допомогою наступної команди.

terraform destroy

Підтверджуємо видалення командою yes і чекаємо.

![](Aspose.Words.cb0b4cf3-caa9-4d6c-b4f2-c0908b76d88d.011.png)

![](Aspose.Words.cb0b4cf3-caa9-4d6c-b4f2-c0908b76d88d.012.png)

В нас не залишилося ні віртуальної машини, ні мережі, тобто terraform виконав видалення вірно і не залишив нічого, чого ми й хотіли.
