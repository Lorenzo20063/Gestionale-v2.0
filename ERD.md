# ERD — Gestionale Pizzeria

## 1. Users / Ruoli / Permessi

**users**  
| Campo            | Tipo             | Note                           |
|-----------------|----------------|--------------------------------|
| id               | bigint PK       | Auto-increment                |
| name             | string          | Nome completo                 |
| email            | string UNIQUE   | Email                         |
| password         | string          | Password hash                 |
| role_id          | bigint FK       | Relazione a `roles`           |
| created_at       | timestamp       |                                |
| updated_at       | timestamp       |                                |

**roles**  
| Campo            | Tipo           | Note                          |
|-----------------|----------------|-------------------------------|
| id               | bigint PK     | Auto-increment               |
| name             | string UNIQUE | admin, cassiere, cuoco...    |
| description      | string NULL   | Opzionale                     |
| created_at       | timestamp     |                               |
| updated_at       | timestamp     |                               |

**permissions**  
| Campo            | Tipo           | Note                          |
|-----------------|----------------|-------------------------------|
| id               | bigint PK     | Auto-increment               |
| name             | string UNIQUE | view_comande, manage_menu... |
| description      | string NULL   | Opzionale                     |
| created_at       | timestamp     |                               |
| updated_at       | timestamp     |                               |

**role_permission (pivot)**  
| Campo            | Tipo        | Note                          |
|-----------------|------------|-------------------------------|
| role_id          | bigint FK  | Referenza `roles.id`          |
| permission_id    | bigint FK  | Referenza `permissions.id`    |

---

## 2. Tavoli / Comande / Piatti

**tables (tavoli)**  
| Campo            | Tipo          | Note                         |
|-----------------|---------------|-------------------------------|
| id               | bigint PK     | Auto-increment               |
| name             | string        | Numero o nome tavolo         |
| seats            | integer       | Numero posti                 |
| status           | enum          | libero, occupato, prenotato |
| created_at       | timestamp     |                               |
| updated_at       | timestamp     |                               |

**orders (comande)**  
| Campo            | Tipo           | Note                               |
|-----------------|----------------|------------------------------------|
| id               | bigint PK      | Auto-increment                     |
| user_id          | bigint FK      | Cameriere / cassiere               |
| table_id         | bigint FK      | Tavolo associato (nullable se take-away) |
| status           | enum           | aperta, in_cucina, servita, chiusa |
| total            | decimal(10,2)  | Totale ordine                       |
| created_at       | timestamp      |                                     |
| updated_at       | timestamp      |                                     |

**order_items (piatti per comanda)**  
| Campo            | Tipo          | Note                                |
|-----------------|---------------|-------------------------------------|
| id               | bigint PK     | Auto-increment                      |
| order_id         | bigint FK     | Referenza `orders.id`               |
| product_id       | bigint FK     | Referenza `products.id`             |
| quantity         | integer       | Quantità                            |
| price            | decimal(10,2) | Prezzo al momento dell'ordine       |
| status           | enum          | ordinato, in_preparazione, pronto  |
| created_at       | timestamp     |                                     |
| updated_at       | timestamp     |                                     |

**products (piatti/menu)**  
| Campo            | Tipo          | Note                                |
|-----------------|---------------|-------------------------------------|
| id               | bigint PK     | Auto-increment                      |
| name             | string        | Nome piatto                          |
| description      | text          |                                     |
| price            | decimal(10,2) | Prezzo standard                     |
| category_id      | bigint FK     | Categoria (antipasti, pizza...)    |
| image_id         | bigint FK     | Media Library                       |
| created_at       | timestamp     |                                     |
| updated_at       | timestamp     |                                     |

**categories (categorie piatti)**  
| Campo            | Tipo           | Note                                |
|-----------------|----------------|-------------------------------------|
| id               | bigint PK      | Auto-increment                      |
| name             | string         | Pizza, Bevande, Dolci               |
| created_at       | timestamp      |                                     |
| updated_at       | timestamp      |                                     |

---

## 3. Magazzino / Stock

**ingredients (ingredienti)**  
| Campo            | Tipo           | Note                                |
|-----------------|----------------|-------------------------------------|
| id               | bigint PK      | Auto-increment                      |
| name             | string         | Nome ingrediente                     |
| unit             | string         | Kg, l, pezzi                         |
| created_at       | timestamp      |                                     |
| updated_at       | timestamp      |                                     |

**product_ingredient (pivot)**  
| Campo            | Tipo           | Note                                |
|-----------------|----------------|-------------------------------------|
| product_id       | bigint FK      | Referenza `products.id`             |
| ingredient_id    | bigint FK      | Referenza `ingredients.id`          |
| quantity         | decimal(10,2)  | Quantità necessaria per il piatto  |

**stock_movements**  
| Campo            | Tipo           | Note                                |
|-----------------|----------------|-------------------------------------|
| id               | bigint PK      | Auto-increment                      |
| ingredient_id    | bigint FK      |                                     |
| quantity         | decimal(10,2)  | Positivo = entrata, Negativo = uscita |
| type             | enum           | carico, scarico, rettifica          |
| created_by       | bigint FK      | User ID                             |
| created_at       | timestamp      |                                     |

---

## 4. Pagamenti / Fatture

**payments**  
| Campo            | Tipo           | Note                                |
|-----------------|----------------|-------------------------------------|
| id               | bigint PK      | Auto-increment                      |
| order_id         | bigint FK      | Comanda associata                   |
| amount           | decimal(10,2)  |                                    |
| method           | enum           | contanti, carta, POS, online       |
| status           | enum           | pending, completed, failed         |
| created_at       | timestamp      |                                     |
| updated_at       | timestamp      |                                     |

**invoices (fatture)**  
| Campo            | Tipo           | Note                                |
|-----------------|----------------|-------------------------------------|
| id               | bigint PK      | Auto-increment                      |
| payment_id       | bigint FK      | Riferimento al pagamento            |
| number           | string UNIQUE  | Numero fattura                      |
| xml_data         | text           | Fatturazione elettronica            |
| created_at       | timestamp      |                                     |
| updated_at       | timestamp      |                                     |

---

## 5. Delivery / Rider / Tracking

**riders**  
| Campo            | Tipo           | Note                                |
|-----------------|----------------|-------------------------------------|
| id               | bigint PK      | Auto-increment                      |
| user_id          | bigint FK      | Referenza `users.id`               |
| vehicle_type     | string         | bici, scooter, auto                 |
| status           | enum           | disponibile, in_consegna, fermo    |
| created_at       | timestamp      |                                     |
| updated_at       | timestamp      |                                     |

**delivery_orders**  
| Campo            | Tipo           | Note                                |
|-----------------|----------------|-------------------------------------|
| id               | bigint PK      | Auto-increment                      |
| order_id         | bigint FK      | Comanda associata                   |
| rider_id         | bigint FK      | Rider assegnato                     |
| status           | enum           | pending, picked_up, delivered      |
| latitude         | decimal(10,6)  | Posizione attuale                    |
| longitude        | decimal(10,6)  | Posizione attuale                    |
| estimated_time   | datetime       | Tempo stimato                        |
| created_at       | timestamp      |                                     |
| updated_at       | timestamp      |                                     |

---

## 6. Media / File

**media**  
| Campo            | Tipo           | Note                                |
|-----------------|----------------|-------------------------------------|
| id               | bigint PK      | Auto-increment                      |
| model_type       | string         | Tipo modello collegato (product, user...) |
| model_id         | bigint          | ID del modello associato            |
| file_name        | string         | Nome file                           |
| file_path        | string         | Path / URL                           |
| type             | string         | immagine, pdf, altro                |
| created_at       | timestamp      |                                     |
| updated_at       | timestamp      |                                     |

---

## 7. Logging / Activity

**activity_log**  
| Campo            | Tipo           | Note                                |
|-----------------|----------------|-------------------------------------|
| id               | bigint PK      | Auto-increment                      |
| log_name         | string         | Tipo log                             |
| description      | text           |                                     |
| subject_type     | string         | Modello interessato                 |
| subject_id       | bigint          | ID modello                          |
| causer_type      | string         | Chi ha effettuato l'azione         |
| causer_id        | bigint          | ID utente                            |
| properties       | json           | Extra info                           |
| created_at       | timestamp      |                                     |

---

## 8. Suggerimenti per estensioni future

- Multi-sede: aggiungere `branches` collegato a `orders`, `products`, `stock`.
- Promozioni e coupon: `promotions` + `order_promotion`.
- Prenotazioni tavoli: `reservations` con FK `table_id` e `user_id`.
- Feedback clienti: `reviews` con rating, commenti, order_id.
- Analisi AI: `analytics_events` per tracciare azioni utente, ordini, stock.
- Gestione allergeni: FK `allergens` collegati a `products`.

---

**Legenda relazioni principali:**

- `users` → `roles` = 1:N  
- `roles` → `permissions` = M:N (pivot `role_permission`)  
- `orders` → `users` = N:1 (cameriere)  
- `orders` → `tables` = N:1  
- `order_items` → `orders` = N:1  
- `order_items` → `products` = N:1  
- `products` → `categories` = N:1  
- `products` → `ingredients` = M:N (pivot `product_ingredient`)  
- `delivery_orders` → `orders` = 1:1  
- `delivery_orders` → `riders` = N:1  
- `media` → `model_type` + `model_id` = polymorfica  
- `activity_log` → polymorfica per tracking qualsiasi modello  


