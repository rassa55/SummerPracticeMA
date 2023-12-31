![[Pasted image 20230718140215.png]]

Главная идея Dropout — вместо обучения одной DNN обучить ансамбль нескольких DNN, а затем усреднить полученные результаты.

Сети для обучения получаются с помощью **исключения** из сети (dropping out) нейронов с вероятностью ![$p$](https://habrastorage.org/getpro/habr/post_images/112/bb3/965/112bb396570b07f26f792e48c3447496.svg), таким образом, вероятность того, что нейрон останется в сети, составляет ![$q=1-p$](https://habrastorage.org/getpro/habr/post_images/3d1/788/975/3d1788975f4e5e1d5ec69b9186460bf8.svg). “Исключение” нейрона означает, что при любых входных данных или параметрах он возвращает 0.  
  
Исключенные нейроны **не вносят свой вклад** в процесс обучения ни на одном из этапов алгоритма обратного распространения ошибки (backpropagation); поэтому исключение хотя бы одного из нейронов равносильно обучению новой нейронной сети.

## Реализация Dropout в Keras

Используемый нами пакет Keras, для построения и обучения НС позволяет применять алгоритм Dropout к любому отдельному слою. Для демонстрации его работы я смоделировал искусственный пример переобучения распознавания цифр. Взял маленькую обучающую выборку в 5000 изображений. Столько же отвел для проверочной. Число нейронов скрытого слоя установил в 300 – это явно много для таких выборок и такой задачи. Неизбежно должны возникнуть проблемы при обучении. Так и происходит. После 50 эпох мы видим расходящиеся графики:

![](https://proproprogs.ru/htm/neural_network/files/dropout-metod-borby-s-pereobucheniem-neyronnoy-seti.files/image015.jpg)

Теперь, применим к скрытому слою из 300 нейронов алгоритм Dropout с параметром p=0,8 (я специально взял его таким большим, чтобы был виден эффект):

model = keras.Sequential([
    Flatten(input_shape=(28, 28, 1)),
    Dense(300, activation='relu'),
    Dropout(0.8),
    Dense(10, activation='softmax') ])

То есть, мы записываем Dropout после слоя, к которому он применяется. Теперь после обучения у нас возникает следующая картина:

![](https://proproprogs.ru/htm/neural_network/files/dropout-metod-borby-s-pereobucheniem-neyronnoy-seti.files/image016.jpg)

Смотрите, здесь качество обучения на проверочной выборке уже не ухудшается и составляет величину, примерно, 0,22. Тогда как в предыдущем случае она почти достигала значения 0,3. Dropout здесь явно сыграл свою положительную роль.

Конечно, это довольно искусственный, гипертрофированный пример, но он наглядно демонстрирует эффект уменьшения степени специализации отдельных нейронов и повышения качества обучения при сохранении общего числа нейронов сети.

## Как работает Dropout

  
Как уже говорилось, Dropout выключает нейроны с вероятностью ![$p$](https://habrastorage.org/getpro/habr/post_images/112/bb3/965/112bb396570b07f26f792e48c3447496.svg) и, как следствие, оставляет их включенными с вероятностью ![$q=1-p$](https://habrastorage.org/getpro/habr/post_images/3d1/788/975/3d1788975f4e5e1d5ec69b9186460bf8.svg).  
  
**Вероятность выключения каждого нейрона одинакова**. Это означает следующее:  
  
При условии, что  
  

- ![$h(x) = xW + b$](https://habrastorage.org/getpro/habr/post_images/4fe/97a/c2b/4fe97ac2b764c07326074f7dcd44b441.svg) — линейная проекция входного ![$d_i$](https://habrastorage.org/getpro/habr/post_images/72e/da0/951/72eda0951aaebfc31c3c417f692d1500.svg)-мерного вектора x на ![$d_h$](https://habrastorage.org/getpro/habr/post_images/f28/05b/220/f2805b2200cc4f7c8e2e5aefc47780bd.svg)-мерное пространство выходных значений;
- ![$a(h)$](https://habrastorage.org/getpro/habr/post_images/944/429/e3f/944429e3fb1350111c128c346e10542a.svg) – функция активации,

  
  
применение Dropout к данной проекции на этапе обучения можно представить как измененную функцию активации:  

![$f(h)=D⊙a(h),$](https://habrastorage.org/getpro/habr/post_images/e9e/d67/2ef/e9ed672ef03d955a80585932c0843bd0.svg)

  
где ![$D=(X_1,⋯,X_{d_h})$](https://habrastorage.org/getpro/habr/post_images/b41/b3a/cd6/b41b3acd6f4fce7e184e223357006919.svg) – ![$d_h$](https://habrastorage.org/getpro/habr/post_images/f28/05b/220/f2805b2200cc4f7c8e2e5aefc47780bd.svg)-мерный вектор случайных величин ![$X_i$](https://habrastorage.org/getpro/habr/post_images/5f2/643/6c9/5f26436c907286ef0a55ffe9da9ee07d.svg), распределенных по закону Бернулли.  
  
![$X_i$](https://habrastorage.org/getpro/habr/post_images/5f2/643/6c9/5f26436c907286ef0a55ffe9da9ee07d.svg) имеет следующее распределение вероятностей:

![$f(k;p)=\begin{equation*}\begin{cases}p, &\mathrm{if}& k=1\\1-p,&\mathrm{if}& k=0\end{cases}\end{equation*},$](https://habrastorage.org/getpro/habr/formulas/bd7/732/1f8/bd77321f8f87fed192068aa65bf82a78.svg)

где ![$k$](https://habrastorage.org/getpro/habr/post_images/839/452/3d1/8394523d1f4bab66a8544cd365388d87.svg) — все возможные выходные значения.  
  
Очевидно, что эта случайная величина идеально соответствует Dropout, примененному к одному нейрону. Действительно, нейрон выключают с вероятностью ![$p=P(k=1)$](https://habrastorage.org/getpro/habr/post_images/32a/cf4/5f5/32acf45f5162f59e4ab1f49f8e38bd7f.svg), в противном случае — оставляют включенным.  
  
Посмотрим на применение Dropout к _i_-му нейрону:  

![$O_i=X_ia(\sum_{k=1}^{d_i}{w_kx_k+b})=\begin{equation*}\begin{cases}a(\sum_{k=1}^{d_i}{w_kx_k+b}), &\mathrm{if}& X_i=1\\0,&\mathrm{if}& X_i=0\end{cases}\end{equation*},$](https://habrastorage.org/getpro/habr/formulas/d98/357/c46/d98357c46c4fce3dc39a2956a95538ea.svg)

  
где ![$P(X_i=0)=p$](https://habrastorage.org/getpro/habr/post_images/22e/d22/8e4/22ed228e43402c343814c8fd0f7adf14.svg).  
  
Так как на этапе обучения нейрон остается в сети (не подвергается выключению) с вероятностью _q_, на этапе тестирования нам необходимо эмулировать поведение ансамбля нейронных сетей, использованного при обучении. Для этого авторы предлагают на этапе тестирования умножить функцию активации на коэффициент _q_. Таким образом,  
  
**На этапе обучения**:![$O_i=X_ia(\sum_{k=1}^{d_i}{w_kx_k+b})$](https://habrastorage.org/getpro/habr/post_images/a12/73e/963/a1273e9639b76635310ba7e20c3bd4d3.svg),  
  
**На этапе тестирования**:![$O_i=qa(\sum_{k=1}^{d_i}{w_kx_k+b})$](https://habrastorage.org/getpro/habr/post_images/e1f/f8a/43e/e1ff8a43e687d9750d7a0d93a6256a2b.svg)  
  

## Обратный (Inverted) Dropout

  
Возможно использовать немного другой подход — **обратный Dropout**. В данном случае мы умножаем функцию активации на коэффициент не во время тестового этапа, а во время обучения.  
  
Коэффициент равен обратной величине вероятности того, что нейрон останется в сети: ![$\frac{1}{1-p}=\frac{1}{q} $](https://habrastorage.org/getpro/habr/post_images/1c9/3ca/0a9/1c93ca0a9a750517c859a2c780d4480e.svg), таким образом,  
  
**На этапе обучения**: ![$O_i=\frac{1}{q}X_ia(\sum_{k=1}^{d_i}{w_kx_k+b})$](https://habrastorage.org/getpro/habr/post_images/f31/282/07a/f3128207a48d2c703b56d4b540c5a97c.svg),  
**На этапе тестирования**: ![$O_i=a(\sum_{k=1}^{d_i}{w_kx_k+b})$](https://habrastorage.org/getpro/habr/post_images/1a7/b3a/c23/1a7b3ac239b125170a81856c4136236b.svg)  
  
Во многих фреймворках для глубокого обучения Dropout реализован как раз в этой модификации, так как при этом необходимо лишь однажды описать модель, а потом запускать обучение и тестирование на этой модели, меняя только параметр (коэффициент Dropout).  
  
В случае прямого Dropout мы вынуждены изменять нейронную сеть для проведения тестирования, так как без умножения на _q_ нейрон будет возвращать значения выше, чем те, которые ожидают получить последующие нейроны; именно поэтому реализация обратного Dropout встречается чаще.

# Идея отсева

Обучение одной глубокой нейронной сети с большими параметрами данных может привести к переобучению. Можем ли мы обучить несколько нейронных сетей с разными конфигурациями в одном наборе данных и принять среднее значение этих прогнозов?

![](https://machinelearningmastery.ru/img/0-297571-388882.png)

Но создать ансамбль нейронных сетей с различными архитектурами и обучить их на практике было бы невозможно. Выпадать на помощь.

Выпадение дезактивирует нейроны случайным образом на каждом этапе обучения, вместо того, чтобы обучать данные в исходной сети, мы обучаем данные в сети с выпавшими узлами. На следующей итерации шага обучения скрытые нейроны, которые деактивируются в результате отсева, изменяются из-за его вероятностного поведения. Таким образом, применяя отсев, т. Е. ... деактивируя отдельные узлы случайным образом во время обучения, мы можем моделировать ансамбль нейронной сети с различными архитектурами.

## Отсев во время обучения

![](https://machinelearningmastery.ru/img/0-625542-956074.png)

В каждой обучающей итерации каждый узел в сети связан с вероятностью p, оставаться ли в сети или отключать его (выбывать) из сети с вероятностью 1-p Это означает, что веса, связанные с узлами, обновлялись только p долей раз, потому что узлы активны только p раз во время обучения.

## Выпадание во время теста

![](https://machinelearningmastery.ru/img/0-371290-165102.png)

Во время тестирования мы рассматриваем исходную нейронную сеть со всеми присутствующими активациями и масштабируем выход каждого узла на значение p. Так как каждый узел активируется только р раз.

![[Pasted image 20230720124137.png]]