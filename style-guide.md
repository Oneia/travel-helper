# Travel-helper guide

<b>Code Style Angular</b>

## Иннициализация приложения

Бойлер плейт на основании angular-cli, при генерации приложения добавить флаги например.
ng new projectName --style=less --skip-tests=true --routing --prefix=haha --skip-git=true
Больше информации <a target="_blank" href="https://github.com/angular/angular-cli/wiki/new">тут</a>
Так же меняем настройки .angular-cli.json, который в корневой папке.
Добавляем в поле "defaults" (последний ключ обычно).
<pre>
  "component": {
        "spec": false,
        "changeDetection": "OnPush"
      }
</pre>
Теперь при создании компонентов не будут генерится автоматически файлы тестов и changeDetection будет в значении OnPush.
Подобные настройки можно поставить на создание всех типов файлов, смотрим <a target="_blank" href="https://github.com/angular/angular-cli/wiki/generate">тут</a>
## 	Core module

1. Тут объявлят single-use (singleton) элементы (компоненты, сервисы).
2. Сюда инжектим(обычно, не всегда) все api services.
3. Тут создаем сервис(если сервис нужен) для кеширования данных см. работа с данными.
4. Тут обьявляем роунтинг для всех модулей через lazyloading
5. Тут обьявляем статические компоненты(header, menu, main-container)

## Share module

1. Котором объявлять элементы (компоненты, директивы, пайпы), которые не связаны с какими-то фишками, а используются повсеместно в разных частях приложения (например, модальные окна, кнопки и т.п.)
2. Компоненты сервисов не имеют, данных идут через in\out. 
3. Важно понимать при каждом инжекте данного модуля создаются новые экземпляры по этой же причине не используем здесь сервисы так как сервисы должны быть синглтон.

## Работа с данными

1. Создаем базовый api.service, модификация может быть разная.
2. Сервисы по работе с api создаем в модуле с которым работаем, но подключаем в core.module. Данные в api сервисах не храним, для этого есть data-service.
3. Все состояние модуля хранится data-service, переключалки, текстовые и другие инпуты тоже хранятся в потоке. В компонентах данные не хранятся.
4. В сервисе хранения состояния модуля есть три обязательных метода: destroy - вызывается в главном компоненте модуля для очистки сервиса, init - вызывается в главном компоненте модуля для загрузки состояния модуля, initInternal - чтоб не делать много вызовов в конструкторе, выносим в отдельный метод. Первые два наследуются от интерфейса, второй приватный для внутренного вызова в конструкторе.
     
     export class Article extends BaseModel<Article> {
     
       public link:   string;
       public pic:    string;
       public source: string;
       public title:  string;
     
       constructor(data: Partial<Article> = {}) {
         super();
     
         const fields: Array<keyof Article> = [
           'link',
           'pic',
           'source',
           'title',
         ];
     
         this.fillAll(data, fields);
       }
     }</pre>
9. Интерфейс начинаем с префикса “I”, например.
<pre>
export interface IDataService {
  init(): void;
  destroy(): void;
}
</pre>


## Общие рекомендации по приложению
1.  Стараться разбивать компоненты на умные/толстые контейнеры (работают с данными, не имеют своих стилей) и глупые/тонкие (данные получают только через inputs от родительских компонентов, имеют собственные стили, отвечают за презентацию и взаимодействие с пользователем). Все компоненты делать через ChangeDetection.OnPush, если надо changeDetection запускаем вручную, для этого есть сервис ChangeDetectionRef.
2.  Для имен компонентов использовать custom-префикс (в секции `prefix` файла `angular-cli.json` в каждом проекте установить собственное значение вместо `app`).
3.  Для каждого компонента устанавливать ChangeDetectionOnPush если есть возможность. ( проверка запускается только при изменении инпутов(), а не на каждый чих.
4.  Все модули через lazyLoading подгружаются. <b>Модули страниц не инжектить в другие модули</b>.
5.  Для непонятных обьектов есть <a href="https://www.npmjs.com/package/typed-object-interfaces">StdObject</a>
6.  Именна свойств в camelCase
7.  Все методы и свойста имеют идентификаторы, (за исключением lifecycles), Обьявление переменной на следующей строке после декоратора.
8.  Во всех методах указываем тип возвращаемого значения, (void тоже)
9. Сначала идут публичные, потом приватные.
10. В конструкторе все сервисы приватные либо protected.
11. Для подписок использовать сниппет/ У меня подобные конструкции забиты в скрипты для веб-шторма, советую сделать так же.
<pre>
 private subs: Subscription[] = [];
  private set sub(sub: Subscription) { this.subs.push(sub); }
</pre>
12. Использовать https://augury.angular.io/ для дебага.
13. Не делаем сложную логику в темлейтах, для этого есть геттеры (get smt)
14. На обязательно импортировать rx операторы в каждый файл, можно использовать для этого общее хранилище


## Cуммируя
<b>Все приложение асинхронно, данные в инпуты идут через пайп async. Все данные хранятся в потоках, которые в свою очередь в сервисе. Не забываем ставить ChangeDetection.onPush.</b>

Переходим к рассмотрению логики сервиса <br>
<pre>
    private usersBase$ = new BehaviorSubject<User[]>([]);
    private _filterGenderSelected$ = new BehaviorSubject<FilterType>(undefined);
    private _searchText$ = new BehaviorSubject<string>('');
    private _activeList$ = new BehaviorSubject<string>('block');
</pre>
Все состояние держится в потоках, это приватные свойста и обновлять мы их можем только через сеттеры. Описание работы BehaviourSubject - <a target="_blank" href="https://github.com/Reactive-Extensions/RxJS/blob/master/doc/api/subjects/behaviorsubject.md">here</a> 
что стоит знать, BehaviorSubject хранит последнее состояние - BehaviorSubject.prototype.getValue(). 
Почему все хранится в потоках? С ними очень легко работать у нас есть множество операторов предоставляемых rxjs для работы с данными.
<a target="_blank" href="https://www.learnrxjs.io/">Здесь</a> их можно изучить. <br>
<b>Наиболее често используемые</b> <br>
<a target="_blank" href="https://www.learnrxjs.io/operators/transformation/combinelatest.html">combineLatest</a> -  для обьеденения <b>действующих</b> потоков, получить последние значения из каждой последовательности при эммите одного из них <br>
<a target="_blank" href="https://www.learnrxjs.io/operators/transformation/forkjoin.html">forkJoin</a> - для обьеденения <b>completed</b> потоков  <br>
<a target="_blank" href="https://www.learnrxjs.io/operators/transformation/map.html">map</a>- аналог Array.prototype.map<br>
<a target="_blank" href="https://www.learnrxjs.io/operators/transformation/mergemap.html">mergeMap</a> - применяется, когда у вас есть Observable, элементы последовательности которого тоже Observable, а вам хочется объединить все в один поток (чтобы все элементы внутренние Observable порождали событие основного). Не путать со switchMap! <br>
<a target="_blank" href="https://www.learnrxjs.io/operators/transformation/switchmap.html">switchMap</a> - делает complete для предыдущего Observable, то есть в данном случае у нас всегда будет только один активный Observable для интервала:<br>
<a target="_blank" href="https://www.learnrxjs.io/operators/transformation/pairwise.html">pairwise</a> возвращает не только текущее значение, но в месте с ним и предыдущее значение последовательности <br>
<a target="_blank" href="https://www.learnrxjs.io/operators/transformation/first.html">first()</a> - вызывает только раз, тоже самое что take(1)<br>
<a target="_blank" href="https://www.learnrxjs.io/operators/transformation/do.html">do</a>- чтоб сделать какие операции без возвращения нового массива, например для емита. <br> 
<a target="_blank" href="https://www.learnrxjs.io/operators/transformation/filter.html">filter</a> - для фильтрирования, да я кеп:) <br>
<a target="_blank" href="https://www.learnrxjs.io/operators/transformation/debouncetime.html">debounceTime</a> - delay для вызова нового значения <br>
<a target="_blank" href="https://www.learnrxjs.io/operators/transformation/topromise.html">toPromise</a> - преобразовует Observable в Promise <br>
 Здесь статья на русском с примерами -<a target="_blank" href="http://stepansuvorov.com/blog/tag/rxjs/">here</a> <br>
<b>Возвращаемся к работе с потоками<b>
Итак задача получить список пользователей
<pre>
  this.sub = this.cacheData.users$
        .subscribe((res: User[]) => {
          this.usersBase$.next(res);
        });
</pre>
Подписываемся на сервис с данными, получаем оттуда значение, и емиттим его в поток пользователей "usersBase$", это первый поток.
На выходе нам нужен не просто список пользователей, у нас еще есть два фильтра по гендеру и строке поиска, эти два значения тоже хранятся в потоках 
<pre>
  private _filterGenderSelected$ = new BehaviorSubject<FilterType>(undefined);
  private _searchText$ = new BehaviorSubject<string>('')
</pre>
Для получения окончательно списка пользователей обьеденяем все три потока в один и проводим операции фильтра.
<pre>
  this.users$ = Observable.combineLatest(
        this.usersBase$,
        this._filterGenderSelected$,
        this._searchText$
      ).map(([users, gender, search]: [User[], FilterType, string]) => {
  
        return users.filter((el: User) => {
          let trigger: boolean = true;
  
          if (gender) {
            const sex = gender === 1 ? 'male' : 'female';
            trigger = el.gender === sex;
          }
  
          if (!trigger) {
            return false;
          }
  
          if (search) {
            trigger = el.name.first.indexOf(search) === 0
              || el.name.last.indexOf(search) === 0
              || el.name.title.indexOf(search) === 0;
          }
  
          return trigger;
        });
      });
</pre>
Список пользователей будет обновлятся каждый раз, как поменяется значение одного из потоков.
Меняется значение фильтров следующем образом
<pre>
    public filterString(search: string): void {
      this._searchText$.next(search);
    }
</pre>
Для прослушки значений в компоненте, делаем публичный Observables и привязываем их к потокам сервиса
<pre>
  this.searchText$ = this._searchText$.asObservable();
  this.filterGenderSelected$ = this._filterGenderSelected$.asObservable();
  this.activeList$ = this._activeList$.asObservable();
</pre>
Появился еще поток activeList$, для переключения вида, поскольку у нас все в потоках, еще тоже храним в потоке, хоть он и не взаимодействует с другими потоками. Хранить все в потоке, хорошая практика.
<br><br><b>Инициализация данных в компонте</b>
<pre>
    this.userData.init();
    this.users = this.userData.users$;
    this.activeList = this.userData.activeList$;
    this.genderSelected = this.userData.filterGenderSelected$;
    this.searchIn = this.userData.searchText$;
</pre>
Первая строка - инициализация сервиса, загрузка данных в сервис.<br>
Второя - пятая строки - инициализация данных в компоненте. Важно, даже search input имеет значение по умолчанию и изначально привязан к потоку searchText.
Это сейчас Observables, от них не нато отписываться, так как подписывается на них в темплейте, через пайп async
<pre>  <
      [testSearchIn]="searchIn | async"
      [toggleViewIn]="activeList | async"
      [genderIn]="genderSelected | async"
    
      (searchText)="filterString($event)"
      (toggleView)="toggleView($event)"
      (addUser)="addUserAction($event)"
      (toggleGender)="filterMale($event)"
      >
  
    < *ngIf="(activeList | async) === 'block'" [users]="users | async">
    
    < *ngIf="(activeList | async) === 'list'" [users]="users | async">
</pre>
В итоге получаем сервис где хранится все состояние приложение и все реактивно.

## Cache Service для работы с API
Данные с сервера зачастую могут использоваться в разные частях приложения, в разных модулях. Чтоб не грузить их каждый раз удобно использовать сервис который бы их кешировал при первом запросе и потом возвращал следующим подписчикам уже закешированные результаты. 
При следующих подписках нам будут возвращаться закешированные данные. Для каждых данных надо создавать свои потоки и свои сеттеры. Использовать по необходимости. Реализация сервиса может отличаться, но смысл тот остается же.

## Ссылки для изучения rxjs
1. <a target="_blank" href="https://gist.github.com/btroncone/a6e4347326749f938510">ngrx</a>  - store постренный на BehaviorSubject, как и у нас, но с reducers, actions и другими элементами редакса. Тут еще можно в принципе посмотреть на релизацию передачи данных и работу BehaviorSubject, как уже говорил у нас так же.<br>
2. <a target="_blank" href="https://coursehunters.net/course/Introduction-to-Reactive-Programming-RxJS-Angel-skiy">Ввод в реактивное программирование</a> ВИДЕО <br>
3. <a target="_blank" href="">RxJs Subjects</a> разбор работы потоков ВИДЕО <br>
4. <a target="_blank" href="https://coursehunters.net/course/udemy-redux-in-angular">Redux</a> ВИДЕО <br>
5. <a target="_blank" href="https://gist.github.com/staltz/868e7e9bc2a7b8c1f754">Пункт два в текстовом формате</a> <br>
6. <a target="_blank" href="https://www.mindomo.com/ru/mindmap/multicasting-45f5bd8534c74b6087eedb0e1336036b">Subjects</a> СХЕМА<br>
7. <a target="_blank" href="https://www.learnrxjs.io/">Learn rxjs and Examples</a><br>
8. <a target="_blank" href="http://reactivex.io/rxjs/manual/overview.html">RxJs official doc</a><br>


## Полезные ссылки для angular
1. <a target="_blank" href="https://www.gitbook.com/book/rangle-io/ngcourse2/details">Rangle.io</a> лучшая книга для изучения по моему имхо.<br>
2. <a target="_blank" href="https://metanit.com/web/angular2/1.1.php">Angular на русском</a><br>
3. <a target="_blank" href="http://typescript-lang.ru/docs/Basic%20Types.html">Typescript на русском</a><br>
4. <a target="_blank" href="https://www.typescriptlang.org/docs/home.html">Typescript official</a><br>
5. <a target="_blank" href="https://blog.angularindepth.com/everything-you-need-to-know-about-change-detection-in-angular-8006c51d206f">ChangeDetection</a><br>
6. <a target="_blank" href="https://habrahabr.ru/post/281449/">Dependecy Injection</a><br>
7. <a target="_blank" href="https://augury.angular.io/">Augury</a> Браузерная консольная утилита для ангуляра<br>
8. <a target="_blank" href="https://www.youtube.com/channel/UCsBjURrPoezykLs9EqgamOA">Angular firebase</a> канал, где много всяких полезностей по работе с firebase <br>
9. <a target="_blank" href="https://github.com/Yonet/Angular-Interview-Questions">Большой список вопросов для интервью</a><br>
10. Если интересна мобильная разработка можно посмотреть на <a target="_blank" href="ionicframework.com/docs">ionic</a>  для небольших приложений он вполне подходит, ИМХО.
