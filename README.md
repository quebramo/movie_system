# Movie Ratings Prediction

Проект прогнозирует рейтинги фильмов для пользователей на основе исторических оценок и информации о жанрах.

---

## Данные

1. **movies.csv** — данные о фильмах:
   - `movieId` — уникальный идентификатор фильма
   - `title` — название фильма
   - `genres` — жанры фильма
   - `ratings_movie_mean` — средний рейтинг фильма на основе всех пользователей

2. **ratings.csv** — данные об оценках:
   - `userId` — уникальный идентификатор пользователя
   - `movieId` — идентификатор фильма
   - `rating` — оценка фильма пользователем
   - Дополнительно создаются средние оценки пользователя (`rating_user_mean`) и временные признаки (`year`, `month`, `day`, `hour`).

---

## Предобработка данных

- Жанры фильмов закодированы через `MultiLabelBinarizer`.
- Данные о фильмах объединены с данными о рейтингах.
- Для каждого пользователя вычислены средние оценки по каждому жанру.
- Создан датафрейм с бинарными признаками жанров и средними рейтингами.

---

## Разделение на тренировочную и тестовую выборки

```python
from sklearn.model_selection import train_test_split

X_train, X_test, y_train, y_test = train_test_split(
    df.drop(["rating", "title"], axis=1), df["rating"], test_size=0.2, random_state=41
)

---

## Рекомендации

Простая модель рекомендаций основана на популярности фильма:

```python
def get_most_popular_rating(X_train, y_train, X_test, userId, movieId):
    movie_data = X_train[X_train["movieId"] == movieId]
    if movie_data.empty:
        return y_train.mean()  # если фильм отсутствует — средний рейтинг всех фильмов
    else:
        return movie_data.iloc[0]["ratings_movie_mean"]  # иначе — средний рейтинг фильма
```

Прогноз для всех пользователей тестовой выборки:
```python
predict = [
    get_most_popular_rating(X_train, y_train, X_test, userId, movieId)
    for userId, movieId in zip(X_test["userId"], X_test["movieId"])
]
```
- Если фильм отсутствует в тренировочной выборке — возвращается средний рейтинг всех фильмов.
- В противном случае прогноз берется как средний рейтинг данного фильма.
