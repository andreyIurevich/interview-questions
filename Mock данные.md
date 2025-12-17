Наиболее популярные библиотеки:

@faker-js/faker

```javascript
import { faker } from '@faker-js/faker';

faker.person.fullName();
faker.internet.email();
faker.company.name();

```

mockjs

```javascript
Mock.mock({
  'users|5': [{
    id: '@id',
    name: '@name',
    age: '@integer(18,60)'
  }]
});
```

для id и ключей - nanoid

```javascript
import { nanoid } from 'nanoid';
```

Для api

MSW (Mock Service Worker)

```javascript
rest.get('/api/users', (req, res, ctx) => {
  return res(ctx.json(mockUsers));
});
```

