Решение задачи удобно оформить с помощью 3 классов.

Класс **DataPreparation** позволяет выполнить все необходимые преобразования над данными для дальнейшего обучения моделей.

Метод extract_reluctant_users исключит из рассмотрения тех пользователей, которые взаимодействовали с товарами слишком малое число раз. Метод drop_rare_items служит для аналогичной цели с той разницей, что отбрасывает слишком редко заказываемые товары.

Метод train_test поделит выборку на обучающую и тестовую части либо по времени, либо по пропорции между выборками.

Метод common_only оставит только те взаимодействия, которые: (1) осуществлены пользователями, сделавшими заказ и в обучающей, и в тестовой частяъ выборки; (2) связаны с товарами, заказанными и в обучающей, и в тестовой выборках. С рациональной точки зрения, оценить качество модели по пользователю, принадлежащему лишь одной выборке, не получится, поэтому их можно отбросить. С технической точки зрения, это снизит вычислительную сложность.

Метод csr_matrix_via_encoder построит из обучающей и тестовой выборок 2 разрезженные матрицы csr-формата, причём пользователи и товары получат одинаковую индексацию в обеих новых матрицах.


Класс **CandidatesExtractor** отвечает за отбор кандидатов в ранжирование. Помимо идентификаторов товаро-кандидатов, на выход получим соответствующие им скоры и ранги, которые и будут фичами для градиентного бустинга, использующегося для ранжирования.

Метод fit отвечает за обучение моделей из библиотеки [LightFM](https://making.lyst.com/lightfm/docs/home.html):

*   Weighted Approximate-Rank Pairwise loss или [WARP](https://static.googleusercontent.com/media/research.google.com/en//pubs/archive/37180.pdf);
*   Bayesian Personalized Ranking или [BPR](https://arxiv.org/ftp/arxiv/papers/1205/1205.2618.pdf);
*   Logistic Matrix Factorization или [LMF](https://web.stanford.edu/~rezab/nips2014workshop/submits/logmat.pdf);
*   k-Order Statistic Weighted Approximate-Rank Pairwise loss или [k-OS WARP](https://static.googleusercontent.com/media/research.google.com/en//pubs/archive/41534.pdf).

Опробованные модели из библиотеки [implicit](https://github.com/benfred/implicit) показали себя хуже. Качество моделей по метрике Recall@20 среди моделей LightFM разнится несильно, поэтому в методе можно обучить все 4 модели. При этом из-за технических ограничений в RAM для отбора кандидатов следует использовать лишь 2 из них.

Метод evaluate используется для оценки модели по 3 метрикам качества: ROC-AUC, Precision@K и Recall@K.

В библиотеке LightFM не предусмотрено структурированное извлечение скоров айтемов. Это позволяет сделать метод calculate_scores. При том, что он работает довольно долго, RAM будет загружаться незначительно.

Метод candidates_extraction отбирает кандидаты. Для воспроизведения результатов на использованных ранее данных можно не запускать вычисления заново, а загрузить и работать с предпосчитанными скорами товаров.


Класс **HybridRecommender** реализует модель 2 этапа, ответственную за ранжирование товаров-кандидатов.

Метод prepare_data преобразовывает данные к требуемому формату, который ожидает на вход бустинг.

Метод train_test разделяет выборку на обучающую и тестовую части.

Метод fit обучает модель градиентного бустинга.

Метод recommend составляет рекомендации на основе обученной модели бустинга.
