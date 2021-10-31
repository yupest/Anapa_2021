```python
import matplotlib.pyplot as plt
import wordcloud
import pandas as pd
from IPython.display import display
```


```python
# функция для рисования облака слов
def get_cloud(words_list, title, cm, path):
    stop = ["который", "кой_случай","большой_желание","этот_магазин","вообще", "которое","которые","много","сказал", "ничего","второй_раз","этот_гостиница","этот_церковь", "этот_автобус", "один_место","этот_храм","свой_дело", "такой_ощущение", "этот_парк", "один_сторона", "свой_время", "один_раз", "первый_раз", "этот_отель","данный_магазин", "другой_магазин","какой", "такой", "тот", "если","только", "есть","была", "место", "месте", "было", "были", "тогда","очень", "этот", "другой", "также", "чтобы", "быть", "мочь", "свой", "весь", "один"]
    if 'позитивные' in title:
        stop.extend(["весь_парк", "этот_парк","особый_фекалька", "мой_дочь", "другой_время", "небольшой_часть", "немногий_место", "городской_суета", "такой_красота", "большой_количество", "огромное_количество", "любой_время", "этот_участок", "этот_набережная", "сам_река", "наш_строитель", "грязный_собака", "выхлопной_газ"])
    elif 'негативн' in title:
        stop.extend(["отличное_место", "хороший_парк","красивый_мост","дневный_прогулка", "самый_чувство", "отличный_место", "никакой_давление", "третий_глаз", "открытый_человек", "открытый_площадка", "маленький_ребенок", "последний_кусочек", "хороший_место", "дневный_прогулка", "самый_чувство", "отличный_место", "никакой_давление", "третий_глаз", "открытый_человек", "открытый-площадка", "маленький_ребенок", "последний_кусочек","этот_парк", "старый_центр", "один_сторона", "такой_ощущение", "парковая_зона", "прекрасное_место", "некоторый_место", "некоторый_аттракцион", "такой_место", "этот_мост", "тихий_место", "сам_центр", "центральный_место", "сам_конструкция", "городской_парк", "детский_площадка", "большой_количество", "огромный_парк", "хороший_тень", "зеленый_зона", "краснодарский_край", "прекрасный_дерево", "сам_дело", "больший_территория", "некоторый_человек", "прохладный_тень", "хороший_расположение", "один_слово","больший_дерево", "каждый_день", "некоторый_место", "больший_часть", "свой_питомец", "редкий_место", "живописный_окрестность", "удобный_дорожка", "мой_сердечко", "прохладный_тень", "прогулочный_зона", "прекрасный место"])
    wc = wordcloud.WordCloud(collocations=True, 
                             background_color = 'white', 
                             width=1000, 
                             height=600, 
                             min_word_length=4, 
                             colormap=cm,
                             stopwords= stop).generate(" ".join(words_list))
    
    plt.figure(figsize=(20,10))
    plt.imshow(wc, interpolation="bilinear")
    plt.title(title)
    plt.axis("off")
    plt.show()
    wc.to_file(path+'/'+title+'.png')
```


```python
g = ['NOUN','ADJF', 'ADJS', 'COMP','VERB', 'INFN', 'PRTF', 'PRTS', 'GRND', 'ADVB']
```

# Общепит


```python
places_df = pd.read_csv('data/restaurants_TA.csv')
df = pd.read_csv('data/reviews_restaurants_TA.csv')
words = pd.read_csv('data/words_restaurants.csv')
bigramms = pd.read_csv('data/bigramms_restaurants.csv')
```

## Анализ слов по всем отзывам


```python
get_cloud(words.loc[( (words.grammema.isin(g)) & (words.rating<3)) , 'normal_form'].to_list(), 'Негативные слова', 'PuBu', 'data/charts')
get_cloud(words.loc[( (words.grammema.isin(g)) & (words.rating>3)) , 'normal_form'].to_list(), 'Позитивные слова', 'OrRd','data/charts')
```


![png](output_6_0.png)



![png](output_6_1.png)



```python
get_cloud(words.loc[( (words.grammema=='NOUN') & (words.rating<3)) , 'normal_form'].to_list(), 'Общепит (негативные слова)', 'PuBu', 'data/charts')
get_cloud(words.loc[( (words.grammema=='NOUN') & (words.rating>3)) , 'normal_form'].to_list(), 'Общепит (позитивные слова)', 'OrRd','data/charts')
```


![png](output_7_0.png)



![png](output_7_1.png)


## Анализ биграмм по всем отзывам


```python
bigramms.shape
```




    (145188, 12)




```python
bigramms = bigramms.drop(columns = 'category').merge(places_df[['Category AN', 'url']], how = 'left', on = 'url').rename(columns = {'Category AN':'category'})
```


```python
bigramms = bigramms.merge(places_df[['url', 'category']])
```


```python
b_temp = bigramms.loc[((~bigramms.word1.str.contains('такой|сам|каждый|тот|целый|другой|один|свой|мой|следующий|данный|этот|весь|самый|наш')) & (bigramms.tag1.isin(['ADJF', 'ADJS'])) & (bigramms.tag2 == 'NOUN'))]
```


```python
get_cloud(b_temp.loc[(~bigramms.word1.str.contains('красивый')) & ((b_temp.rating<3) | ((b_temp.rating==3) & (b_temp.score<0))), 'bigramm'].str[2:-2].str.split("', '").str.join('_').to_list(), 'Общепит (негативные биграммы)', 'Blues','data/charts')
get_cloud(b_temp.loc[(b_temp.rating>3) , 'bigramm'].str[2:-2].str.split("', '").str.join('_').to_list(), 'Общепит (позитивные биграммы)', 'Oranges','data/charts')
```


![png](output_13_0.png)



![png](output_13_1.png)


## Анализ биграмм по категориям


```python
for i in b_temp.category.drop_duplicates():
    try:
        get_cloud(b_temp.loc[(b_temp.category == i) & (~bigramms.word1.str.contains('красивый')) & ((b_temp.rating<3) | ((b_temp.rating==3) & (b_temp.score<0))), 'bigramm'].str[2:-2].str.split("', '").str.join('_').to_list(), i+ ' (негативные биграммы)', 'Blues','data/charts')
        get_cloud(b_temp.loc[(b_temp.category == i) & (b_temp.rating>3) , 'bigramm'].str[2:-2].str.split("', '").str.join('_').to_list(), i+' (позитивные биграммы)', 'Oranges','data/charts')
    except:
        print(i)
```


![png](output_15_0.png)



![png](output_15_1.png)



![png](output_15_2.png)



![png](output_15_3.png)



![png](output_15_4.png)



![png](output_15_5.png)



![png](output_15_6.png)



![png](output_15_7.png)



![png](output_15_8.png)



![png](output_15_9.png)



![png](output_15_10.png)



![png](output_15_11.png)



![png](output_15_12.png)



![png](output_15_13.png)



![png](output_15_14.png)



![png](output_15_15.png)



![png](output_15_16.png)



![png](output_15_17.png)



![png](output_15_18.png)



![png](output_15_19.png)



![png](output_15_20.png)



![png](output_15_21.png)


    Винный бар
    


![png](output_15_23.png)



![png](output_15_24.png)



![png](output_15_25.png)



![png](output_15_26.png)



![png](output_15_27.png)



![png](output_15_28.png)



![png](output_15_29.png)



![png](output_15_30.png)



![png](output_15_31.png)



![png](output_15_32.png)



![png](output_15_33.png)



![png](output_15_34.png)



![png](output_15_35.png)



![png](output_15_36.png)



![png](output_15_37.png)



![png](output_15_38.png)



![png](output_15_39.png)



![png](output_15_40.png)



![png](output_15_41.png)



![png](output_15_42.png)



![png](output_15_43.png)



![png](output_15_44.png)



```python

```
