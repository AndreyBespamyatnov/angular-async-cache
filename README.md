# angular-async-cache
A simple utility service to help with caching of promises and observables for angular 2.0+

Sample usage

```typescript
import { Http } from '@angular/http';
import { AsyncCache, LocalStorageStrategy, InMemoryStrategy, AsyncCacheModule } from 'angular-async-cache';

@Injectable()
class CarService {

  constructor(
    private http: Http, 
    private asyncCache: AsyncCache, 
    private localStorageStrategy: LocalStorageStrategy
  ) {}
  
  getCars(): Observable<Car[]> {
  
    const cars$: Observable<Car[]> = this.http.get('/cars').map(res => res.json());
    return asyncCache.create(cars$, {
      cacheStrategy: this.localStorageStrategy, // cache the data in localstorage
      fromCacheAndReplay: true // this is the special sauce - first emit the data from localstorage, then re-fetch the live data from the API and emit a second time. The async pipe will then re-render and update the UI
    });
  
  }

}

@Component({
  template: `
    <div *ngFor="car of cars | async">
      {{ car.model }}
    </div>
  `
})
class MyComponent {

  cars: Observable<Car[]>;

  constructor(carService: CarService) {
    this.cars = carService.getCars();
  }

}

@NgModule({
  imports: [
    AsyncCacheModule.forRoot({
      cacheStrategy: new LocalStorageStrategy(),
      fromCacheAndReplay: true
    })
  ]
})
class MyModule {}

```

Interfaces
```typescript
interface CacheStrategy {
  
  has(key: string): Observable<any>;

  get(key: string): Observable<any>;

  set(key: string, value: any): Observable<any>;
  
  delete(key: string): Observable<any>;
  
  clear(): Observable<any>;
  
  keys(): Observable<any>;

}

type GetAsyncValueFunction = () => Observable<any>|Promise<any>;

interface AsyncCache {

  create(getAsyncValue: GetAsyncValueFunction | Observable<any>, {cacheStrategy, fromCacheAndReplay}: {cacheStrategy: CacheStrategy, fromCacheAndReplay: boolean}) {}

}

```
