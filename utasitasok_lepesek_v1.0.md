# Iskola – Angular CRUD projekt (Utasítások & Lépések v1.0)

Forrás: *Utasítások&Lépések_v1.0.pdf* fileciteturn0file0

## 1) Alap lépések

### Új projekt létrehozása (CSS + Routing)
```bash
ng new iskola --style=css --routing=true
```
Válaszok:
- SSR & SSG – **NO**
- Zoneless application – **NO**
- AI Tools – **NO**

### Belépés a projekt mappába
> **Addig ne adj ki más parancsot, amíg ezt meg nem tetted!**
```bash
cd iskola
```

## 2) Bootstrap beállítása

### Telepítés
```bash
npm install bootstrap
```

### `angular.json` módosítása
A `styles` tömbbe vedd fel:
```json
"styles": [
  "node_modules/bootstrap/dist/css/bootstrap.min.css",
  "src/styles.css"
]
```

## 3) Felesleges elemek eltávolítása

### `src/app/app.html`
Töröld a felesleges részeket, **csak ez maradjon**:
```html
<router-outlet />
```

## 4) HTTP kérés kezelése

### `src/app/app.config.ts`
Add hozzá a `provideHttpClient()` sort a `providers` tömbhöz:

```ts
import { ApplicationConfig, provideBrowserGlobalErrorListeners, provideZoneChangeDetection } from '@angular/core';
import { provideRouter } from '@angular/router';
import { routes } from './app.routes';
import { provideHttpClient } from '@angular/common/http';

export const appConfig: ApplicationConfig = {
  providers: [
    provideBrowserGlobalErrorListeners(),
    provideZoneChangeDetection({ eventCoalescing: true }),
    provideRouter(routes),
    provideHttpClient() // Ezzel bővítjük, BE KELL GÉPELNI!
  ]
};
```

## 5) Komponensek létrehozása (terminál parancsok)

### Service létrehozása
```bash
ng generate service services/api
# vagy röviden:
ng g s services/api
```

### Model fájlok létrehozása
```bash
ng generate interface models/diak
ng generate interface models/osztaly
```

### CRUD komponensek létrehozása
```bash
# Diákok listája
ng generate component components/diakok-lista.component

# Új diák felvétele
ng generate component components/diak-letrehozas.component

# Diák szerkesztése
ng generate component components/diak-szerkesztes.component

# Hiba oldal
ng generate component components/hiba
```

## 6) Service tartalom elkészítése

### `src/app/services/api.ts`
```ts
import { Injectable } from '@angular/core';
import { HttpClient } from '@angular/common/http';
import { Observable } from 'rxjs';

@Injectable({
  providedIn: 'root'
})
export class ApiService {
  private readonly baseUrl = 'http://localhost/iskola';

  constructor(private http: HttpClient) {}

  // -----------------------
  // OSZTÁLYOK (READ)
  // -----------------------
  getOsztalyok(): Observable<any> {
    return this.http.get<any>(`${this.baseUrl}/osztalyok_read.php`);
  }

  // -----------------------
  // DIÁKOK (CRUD)
  // -----------------------
  getDiakok(): Observable<any> {
    return this.http.get<any>(`${this.baseUrl}/tanulok_read.php`);
  }

  createDiak(payload: any): Observable<any> {
    return this.http.post<any>(`${this.baseUrl}/tanulok_create.php`, payload);
  }

  updateDiak(payload: any): Observable<any> {
    return this.http.put<any>(`${this.baseUrl}/tanulok_update.php`, payload);
  }

  deleteDiak(id: number): Observable<any> {
    return this.http.delete<any>(`${this.baseUrl}/tanulok_delete.php?id=${id}`);
  }
}
```

> Megjegyzés: a szerkesztő komponensben szerepel egy `getDiakById(...)` hívás. Ehhez külön végpont + service metódus szükséges (ha a backend ad ilyen read-by-id PHP-t).

## 7) Modellek elkészítése

### `src/app/models/diak.ts`
```ts
export interface Diak {
  id: number;
  vezeteknev: string;
  keresztnev: string;
  email: string;
  osztaly_id: number;
  osztaly_nev: string;
  szuletesi_datum: string | null;
}
```

### `src/app/models/osztaly.ts`
```ts
export interface Osztaly {
  id: number;
  nev: string;
}
```

## 8) Tanulók listázása & törlése

### `src/app/components/diakok-lista.component/diakok-lista.component.ts`
> Gyakori hiba: **CommonModule** importálása kell az `*ngFor`-hoz.

```ts
import { Component, OnInit } from '@angular/core';
import { ApiService } from '../../services/api';
import { CommonModule } from '@angular/common';
import { RouterLink } from '@angular/router';

@Component({
  selector: 'app-diakok-lista',
  standalone: true,
  imports: [CommonModule, RouterLink],
  templateUrl: './diakok-lista.component.html',
  styleUrls: ['./diakok-lista.component.css']
})
export class DiakokListaComponent implements OnInit {
  diakok: any[] = [];
  loading = false;
  error = '';

  constructor(private api: ApiService) {}

  ngOnInit(): void {
    this.loadDiakok();
  }

  loadDiakok(): void {
    this.loading = true;
    this.error = '';

    this.api.getDiakok().subscribe({
      next: (res) => {
        this.diakok = res.data;
        this.loading = false;
      },
      error: () => {
        this.error = 'Hiba történt a diákok betöltésekor.';
        this.loading = false;
      }
    });
  }

  // Tanuló törlése, ehhez nem kell külön komponens
  deleteDiak(id: number): void {
    if (!confirm('Biztosan törlöd a kiválasztott diákot?')) return;

    this.api.deleteDiak(id).subscribe({
      next: () => this.loadDiakok(),
      error: () => alert('Hiba történt a törlés során.')
    });
  }
}
```

### `src/app/components/diakok-lista.component/diakok-lista.component.html`
```html
<div class="container mt-4">
  <h2 class="mb-3">Diákok listája</h2>

  <table class="table table-striped table-bordered table-hover">
    <thead class="table-dark">
      <tr>
        <th>#</th>
        <th>Vezetéknév</th>
        <th>Keresztnév</th>
        <th>Email</th>
        <th>Osztály</th>
        <th>Születési dátum</th>
        <th style="width: 160px;">Műveletek</th>
      </tr>
    </thead>

    <tbody>
      <tr *ngFor="let d of diakok; let i = index">
        <td>{{ i + 1 }}</td>
        <td>{{ d.vezeteknev }}</td>
        <td>{{ d.keresztnev }}</td>
        <td>{{ d.email }}</td>
        <td>{{ d.osztaly_nev }}</td>
        <td>{{ d.szuletesi_datum }}</td>
        <td>
          <button class="btn btn-sm btn-warning me-2">
            Szerkesztés
          </button>
          <button class="btn btn-sm btn-danger" (click)="deleteDiak(d.id)">
            Törlés
          </button>
        </td>
      </tr>
    </tbody>
  </table>
</div>
```

## 9) Routing – listázás kezdőoldalnak

### `src/app/app.routes.ts`
```ts
import { Routes } from '@angular/router';
import { DiakokListaComponent } from './components/diakok-lista.component/diakok-lista.component';

export const routes: Routes = [
  { path: '', component: DiakokListaComponent }
];
```

## 10) Új tanuló létrehozása

### `src/app/components/diak-letrehozas.component/diak-letrehozas.component.ts`
> Fontos: **CommonModule** és **FormsModule**.

```ts
import { Component, OnInit } from '@angular/core';
import { CommonModule } from '@angular/common';
import { ApiService } from '../../services/api';
import { Osztaly } from '../../models/osztaly';
import { FormsModule } from '@angular/forms';

@Component({
  selector: 'app-diak-letrehozas',
  standalone: true,
  imports: [CommonModule, FormsModule],
  templateUrl: './diak-letrehozas.component.html',
  styleUrls: ['./diak-letrehozas.component.css']
})
export class DiakLetrehozasComponent implements OnInit {
  osztalyok: Osztaly[] = [];

  form = {
    vezeteknev: '',
    keresztnev: '',
    email: '',
    osztaly_id: 0,
    szuletesi_datum: '' // üres string ok, backend kezelheti null-ként
  };

  constructor(private api: ApiService) {}

  ngOnInit(): void {
    this.loadOsztalyok();
  }

  loadOsztalyok(): void {
    this.api.getOsztalyok().subscribe({
      next: (res) => (this.osztalyok = res.data),
      error: () => alert('Nem sikerült betölteni az osztályokat.')
    });
  }

  createDiak(): void {
    if (!this.form.vezeteknev || !this.form.keresztnev || !this.form.email || this.form.osztaly_id <= 0) {
      alert('Töltsd ki a kötelező mezőket!');
      return;
    }

    const payload = {
      vezeteknev: this.form.vezeteknev.trim(),
      keresztnev: this.form.keresztnev.trim(),
      email: this.form.email.trim(),
      osztaly_id: Number(this.form.osztaly_id),
      szuletesi_datum: this.form.szuletesi_datum.trim() === '' ? null : this.form.szuletesi_datum.trim()
    };

    this.api.createDiak(payload).subscribe({
      next: () => {
        alert('Sikeres mentés!');
        this.form = { vezeteknev: '', keresztnev: '', email: '', osztaly_id: 0, szuletesi_datum: '' };
      },
      error: () => alert('Hiba történt a mentés során (email lehet, hogy már foglalt).')
    });
  }
}
```

### `src/app/components/diak-letrehozas.component/diak-letrehozas.component.html`
```html
<div class="container mt-4">
  <div class="card bg-dark text-light border-secondary">
    <div class="card-body">
      <h2 class="card-title mb-4">Új diák felvétele</h2>

      <form (ngSubmit)="createDiak()">
        <!-- Vezetéknév + Keresztnév -->
        <div class="row g-3 mb-3">
          <div class="col-md-6">
            <div class="form-floating">
              <input
                type="text"
                class="form-control"
                id="vezeteknev"
                placeholder="Vezetéknév"
                required
                name="vezeteknev"
                [(ngModel)]="form.vezeteknev"
              />
              <label for="vezeteknev">Vezetéknév</label>
            </div>
          </div>

          <div class="col-md-6">
            <div class="form-floating">
              <input
                type="text"
                class="form-control"
                id="keresztnev"
                placeholder="Keresztnév"
                required
                name="keresztnev"
                [(ngModel)]="form.keresztnev"
              />
              <label for="keresztnev">Keresztnév</label>
            </div>
          </div>
        </div>

        <!-- Email -->
        <div class="form-floating mb-3">
          <input
            type="email"
            class="form-control"
            id="email"
            placeholder="Email"
            required
            name="email"
            [(ngModel)]="form.email"
          />
          <label for="email">Email</label>
        </div>

        <!-- Osztály -->
        <div class="form-floating mb-3">
          <select
            class="form-select"
            id="osztaly"
            required
            name="osztaly_id"
            [(ngModel)]="form.osztaly_id"
          >
            <option value="" disabled>Válassz osztályt</option>
            <option *ngFor="let o of osztalyok" [value]="o.id">
              {{ o.nev }}
            </option>
          </select>
          <label for="osztaly">Osztály</label>
        </div>

        <!-- Születési dátum -->
        <div class="form-floating mb-4">
          <input
            type="date"
            class="form-control"
            id="szuletesiDatum"
            placeholder="Születési dátum"
            name="szuletesi_datum"
            [(ngModel)]="form.szuletesi_datum"
          />
          <label for="szuletesiDatum">Születési dátum</label>
        </div>

        <!-- Mentés -->
        <button type="submit" class="btn btn-primary w-100">Mentés</button>
      </form>
    </div>
  </div>
</div>
```

## 11) Navigációs sáv

### `src/app/app.html`
```html
<!-- Felső navigáció -->
<nav class="navbar navbar-expand navbar-dark bg-dark">
  <div class="container-fluid">
    <span class="navbar-brand">Iskola – Diákok</span>

    <div class="ms-auto">
      <a class="btn btn-primary" routerLink="/diak-letrehozas">
        + Új diák
      </a>
    </div>
  </div>
</nav>

<!-- Oldaltartalom -->
<div class="container-fluid mt-3">
  <router-outlet></router-outlet>
</div>
```

### RouterLink import az `app.ts`-ben
A navigáció működéséhez:
```ts
imports: [RouterOutlet, RouterLink],
```

## 12) Meglévő tanuló szerkesztése

### `src/app/components/diak-szerkesztes.component/diak-szerkesztes.component.ts`
```ts
import { Component, OnInit } from '@angular/core';
import { CommonModule } from '@angular/common';
import { FormsModule } from '@angular/forms';
import { ActivatedRoute, Router } from '@angular/router';
import { ApiService } from '../../services/api';
import { Osztaly } from '../../models/osztaly';

@Component({
  selector: 'app-diak-szerkesztes',
  standalone: true,
  imports: [CommonModule, FormsModule],
  templateUrl: './diak-szerkesztes.component.html',
  styleUrls: ['./diak-szerkesztes.component.css']
})
export class DiakSzerkesztesComponent implements OnInit {
  diakId = 0;
  osztalyok: Osztaly[] = [];

  form = {
    vezeteknev: '',
    keresztnev: '',
    email: '',
    osztaly_id: 0,
    szuletesi_datum: ''
  };

  constructor(
    private api: ApiService,
    private route: ActivatedRoute,
    private router: Router
  ) {}

  ngOnInit(): void {
    this.diakId = Number(this.route.snapshot.paramMap.get('id'));
    if (!this.diakId) {
      alert('Hiányzó diák azonosító.');
      this.router.navigate(['/']);
      return;
    }

    this.loadOsztalyok();
    this.loadDiak();
  }

  loadOsztalyok(): void {
    this.api.getOsztalyok().subscribe({
      next: (res) => (this.osztalyok = res.data),
      error: () => alert('Nem sikerült betölteni az osztályokat.')
    });
  }

  loadDiak(): void {
    // Ehhez kell: api.getDiakById(...)
    this.api.getDiakById(this.diakId).subscribe({
      next: (res) => {
        const d = res.data;
        this.form = {
          vezeteknev: d.vezeteknev,
          keresztnev: d.keresztnev,
          email: d.email,
          osztaly_id: d.osztaly_id,
          szuletesi_datum: d.szuletesi_datum ?? ''
        };
      },
      error: () => {
        alert('A diák adatai nem tölthetők be.');
        this.router.navigate(['/']);
      }
    });
  }

  updateDiak(): void {
    if (!this.form.vezeteknev || !this.form.keresztnev || !this.form.email || this.form.osztaly_id <= 0) {
      alert('Töltsd ki a kötelező mezőket!');
      return;
    }

    const payload = {
      id: this.diakId,
      vezeteknev: this.form.vezeteknev.trim(),
      keresztnev: this.form.keresztnev.trim(),
      email: this.form.email.trim(),
      osztaly_id: Number(this.form.osztaly_id),
      szuletesi_datum: this.form.szuletesi_datum === '' ? null : this.form.szuletesi_datum
    };

    this.api.updateDiak(payload).subscribe({
      next: () => {
        alert('Sikeres módosítás.');
        this.router.navigate(['/']);
      },
      error: () => alert('Hiba történt a módosítás során.')
    });
  }
}
```

### `src/app/components/diak-szerkesztes.component/diak-szerkesztes.component.html`
(A PDF-ben található sablon alapján.)

## 13) Lista bővítése szerkesztés funkcióval

A listában a „Szerkesztés” gomb legyen routerLink-es:
```html
<button class="btn btn-sm btn-warning me-2" [routerLink]="['/diak-szerkesztes', d.id]">
  Szerkesztés
</button>
```

Routing bővítése:
```ts
{ path: 'diak-szerkesztes/:id', component: DiakSzerkesztesComponent }
```

> Gyakori hiba: a listázó komponensnél is kell a `RouterLink` import (ha még nincs).

## 14) Hiba (404) komponens

### `src/app/components/hiba/hiba.html`
```html
<div class="alert alert-danger" role="alert">
  Hiba! A keresett oldal nem található!
</div>
```

### `routes` bővítése
```ts
// Hiba komponens meghívása, amennyiben rossz útvonalat adnánk meg!
{ path: '**', component: Hiba }
```

## 15) Teszt

```bash
ng serve
```

Nézd meg, hogy minden funkció üzemképes-e.
