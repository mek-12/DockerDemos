# Dotnet sdk 5:0 ile oluşturulan web_api projesini contanierize etme

> Bu demo linux debian buster üzerinden benimde başlangıç olarak docker a giriş amaçlı bir web_api uygulamasını nasıl containerize edebileceğim üzerine, edindiğim bazı tecrübeler ile birlikte bilgilerimi payşalacağım bir referans olacaktır.

> Öncelikle [dotnet new](https://docs.microsoft.com/en-us/dotnet/core/tools/dotnet-new#webapi) ile adını ve özelliklerini kısıtladığım (https olmasın mesela) yeni bir web api uygulaması hazırlıyorum.
```console
 dotnet new webapi --name demo_api_nohttps --no-https
```
> cd ile projenin main directory sine gidiyorum

```console
cd ./demo_api_nohttps
```
> dotnet projesi içerisinde olduğumdan ötürü dotnet build veya dotnet run komutlarını çalıştırabilirim.
dotnet run ile build olduğunu görüyor iseniz sdk eksiksizdir. Sonraki aşamaya geçebiliriz :)

## Projenin main directory sine docker image i oluşturmak için dockerfile dosyasını ekleme

> Oluşturulan projemizi container olarak olarak çalışması için öncelikle container ımızı readonly olarak kendisinden oluşturulmasın olanak sağlayacak image i oluşturmamız gerekir. Bunun için öncelikle [dockerfile](https://docs.docker.com/engine/reference/builder/) dosyasını main directory de tanımlıyoruz.

* FROM mcr.microsoft.com/dotnet/sdk:5.0 AS build
* WORKDIR /source
* 
* COPY *.sln .
* COPY *.csproj .
* RUN dotnet restore 
* 
* COPY . ./demo_api/
* WORKDIR /source/demo_api
* RUN dotnet publish -c release -o /app --no-restore
* 
* FROM mcr.microsoft.com/dotnet/aspnet:5.0-buster-slim-amd64
* WORKDIR /app
* COPY --from=build /app ./
* ENTRYPOINT ["dotnet", "demo_api_nohttps.dll"]

> FROM ile sdk image imizi belirtiyoruz. Dockerfile references da belirtildiği gibi DockerFile içeriği kesinlikle From ile başlamadılır.
Image ı oluşturmak için önclikle layer adın verilen build, release aşamalaını temsil eden katmanları dockerfile da belirtiyoruz. Root dizininde source adında bir workdir belirliyoruz. 

COPY <source_directory> <destinition_directory> ile source klasörüne sln ve csproj dosyalarımızı sırası ile proje dosyamızdan kopyalıyoruz.
(NOT: Workdir ile belirtilen directory baz alınarak COPY, RUN vb. komutlar bu directory için çalışır.)

Ardından /source içerisinde RUN ile dotnet restore ile gerekli nuget paketlerini indiriyoruz.

COPY ile /source folder ı altında /source/demo_api/ directory içerisine proje dosyamızın içierisnde yer alan tüm dosyaları kopyalıyoruz.
RUN komutu ile /app folder ı içerisine derlenmiş dll dosyalarını gönderiyoruz.

WORKDIR ile app/ folderını belirledikten sonra 
ENTRYPOINT ile dotnet in çalışacağı proje lokasyonu belirtilier ve container ayağa kalktığında burada dll i çalıştırır.

## image i build etme

```console
docker build -t mek12/demo_webapi_nohttps -f dockerfile .
```
> ile image oluşturulur.
> -t ile tag ile birlikte image in adı belirtilir. -f ile dockerfile belirtilmiştir.

## Docker image inden container oluşturup -p ile container a istek atılacak portu belirleme

```console
docker run 5000:80 mek12/demo_webapi_nohttps
```
> ile container ımız image den oluşturulmuş ve ayağa kalkmış bulunmaktadır.
> http://localhost:5000/weatherforecast controller ına get isteğinde bulunularak test edebilirsiniz.


Eksik kısımlar doldurulacaktır :)