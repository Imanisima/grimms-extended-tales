## Grimms' Extended Tales ~ N-Gram Model
Using Markov assumption, we will build a basic language model using trigrams of the Gutenberg Corpus using the ```nltk``` package. Specifically, we will be looking at Grimms' Fairy Tales.


```python
import nltk
from nltk.corpus.reader.plaintext import PlaintextCorpusReader

from nltk import bigrams, trigrams
from collections import defaultdict, Counter
import random
```


```python
# placeholder for the n-gram model
n_gram_model = defaultdict(lambda: defaultdict(lambda: 0))
```


```python
# create a new corpus from the Grimms Fairy Tales text file.
corpus_root = 'grimm/'
grimm_txt = PlaintextCorpusReader(corpus_root, '.*')
grimm_txt.words('grimm.txt')
```




    ['FAIRY', 'TALES', 'By', 'The', 'Brothers', 'Grimm', ...]




```python
# calculate frequency for each combination trigram
for sentence in grimm_txt.sents():
    for w1, w2, w3 in trigrams(sentence, pad_right=True, pad_left=True):
        n_gram_model[(w1, w2)][w3] += 1
```


```python
# count probabilities
for w1_w2 in n_gram_model:
    total_count = float(sum(n_gram_model[w1_w2].values()))
    for w3 in n_gram_model[w1_w2]:
        n_gram_model[w1_w2][w3] /= total_count
```


```python
# predict next word
dict(n_gram_model["before", "the"])
```




    {'king': 0.1951219512195122,
     'court': 0.024390243902439025,
     'fairy': 0.04878048780487805,
     'warm': 0.024390243902439025,
     'house': 0.04878048780487805,
     'dog': 0.04878048780487805,
     'princesses': 0.024390243902439025,
     'gate': 0.024390243902439025,
     'cat': 0.04878048780487805,
     'squirrel': 0.024390243902439025,
     'sun': 0.024390243902439025,
     'children': 0.024390243902439025,
     'hut': 0.024390243902439025,
     'fire': 0.024390243902439025,
     'mayor': 0.07317073170731707,
     'witch': 0.024390243902439025,
     'old': 0.024390243902439025,
     'sight': 0.024390243902439025,
     'barrel': 0.024390243902439025,
     'end': 0.024390243902439025,
     'morrow': 0.024390243902439025,
     'hall': 0.024390243902439025,
     'judge': 0.024390243902439025,
     'little': 0.024390243902439025,
     'day': 0.024390243902439025,
     'clock': 0.04878048780487805,
     'palace': 0.024390243902439025}



__Note__: The highest possible word probability wise is ```'king': 0.195```. Let's see what we get next!


```python
dict(n_gram_model["the", "king"])
```




    {'in': 0.008620689655172414,
     'said': 0.03879310344827586,
     '.': 0.05172413793103448,
     'the': 0.017241379310344827,
     ',': 0.125,
     'was': 0.02586206896551724,
     ';': 0.017241379310344827,
     'their': 0.004310344827586207,
     "'": 0.2974137931034483,
     'could': 0.004310344827586207,
     'and': 0.034482758620689655,
     'hoped': 0.004310344827586207,
     'made': 0.01293103448275862,
     'ordered': 0.02586206896551724,
     'with': 0.004310344827586207,
     'asked': 0.008620689655172414,
     'all': 0.004310344827586207,
     'called': 0.004310344827586207,
     '?': 0.004310344827586207,
     'what': 0.004310344827586207,
     'of': 0.017241379310344827,
     'came': 0.02586206896551724,
     'soon': 0.004310344827586207,
     'saw': 0.01293103448275862,
     'felt': 0.004310344827586207,
     'insisted': 0.004310344827586207,
     'replied': 0.004310344827586207,
     'if': 0.004310344827586207,
     'had': 0.034482758620689655,
     'sent': 0.017241379310344827,
     'his': 0.004310344827586207,
     'never': 0.004310344827586207,
     'as': 0.008620689655172414,
     'mourned': 0.004310344827586207,
     'put': 0.004310344827586207,
     'then': 0.004310344827586207,
     'faithfully': 0.008620689655172414,
     'who': 0.004310344827586207,
     'did': 0.01293103448275862,
     'again': 0.008620689655172414,
     'let': 0.004310344827586207,
     'has': 0.008620689655172414,
     'shut': 0.004310344827586207,
     ':': 0.008620689655172414,
     'believe': 0.004310344827586207,
     'to': 0.021551724137931036,
     'thought': 0.004310344827586207,
     'held': 0.004310344827586207,
     'got': 0.008620689655172414,
     'heard': 0.004310344827586207,
     'returned': 0.004310344827586207,
     'must': 0.004310344827586207,
     'looked': 0.008620689655172414,
     'would': 0.004310344827586207,
     'went': 0.004310344827586207,
     'better': 0.004310344827586207,
     'danced': 0.004310344827586207}



__Note:__ As you can see, the highest probability here is ```"'": 0.2974137931034483```. We can assume that the word will be ```king's```.


```python
dict(n_gram_model["king", "'"])
```




    {'s': 1.0}



__Note:__ If we continue on, we will eventually make a pretty understandable sentence. Let's automate this a bit and see what happens!


```python
# save output to file and read
def append_file(text, title):
    with open(title, "a") as f:
        f.write(' '.join([t for t in text if t]))
        f.write('\n')
        f.write('\n')
        f.close()
```


```python
# read file
def read_file(file):
    read_file = open(file, 'r') 
    print(read_file.read()) 
    read_file.close()
```


```python
# use model to create new text
def generate_text(text):
    num_sentences = 0
    n = 5

    while num_sentences < n:

        # select a random probability threshold  
        r = random.random()
        accumulator = 0.002

        for word in n_gram_model[tuple(text[-2:])].keys():
            accumulator += n_gram_model[tuple(text[-2:])][word]

        # select words that are above the probability threshold
            if accumulator >= r:
                text.append(word)
                break

        if text[-2:] == [None, None]:
            num_sentences = num_sentences + 1

    return text
```


```python
# generate a random title
random_words = random.choice(list(n_gram_model.items()))
random_title = [random_words[0][0], random_words[0][1]] # returns a tuple so return string only!
title = 'tales/' + '_'.join([w for w in random_title if w]) + '.txt'
title
```




    'tales/the_throne.txt'




```python
# generate grimm new tales text
paragraph = 0
while paragraph < 6:
    # begin with randomized words from the corpus
    random_entry = random.choice(list(n_gram_model.items()))
    text = [random_entry[0][0], random_entry[0][1]]
    story = generate_text(text)
    
    # add results to text file
    append_file(story, title)
    
    paragraph += 1

# results
read_file(title)

```

    Frederick said , ' You have deceived me !' On the third , he will be true to him , and full of peas , and on till he had done very little trouble about that ; I am quite innocent . But the prince , and ask if it were eating . ' There is some enchantment ; you seem to bind me fast , and when the first ; ' come along with him into a thorn - bush with one rose on it ?' look to the rack , and take leave of the water .

```
obedient and industrious , and as they were sad . Then he was doing so , you may make himself a fire , that she fell on the ground beside a straw , however , you know how to shudder .' But Hansel comforted her and then these two , it will bring down everything I shoot at him . As he had , you should have his revenge . But the mother , ' have I , or bridge in sight , Take us out , ' what does she want now ?'

dry bread together ; and thirdly , I will sleep first ? And thus she had gone forth , no , not knowing where the two brothers who were to cut fuel , he was thankful to the ground , and had neither stairs nor door , and knew not whither , till the judge condemned him to his own . ' Alas !' One of you something in exchange for those three things is true .' But when she unwrapped the cloth it was the matter ; I should like that of an eye , and drank till his loins hurt , and cook and wash and knit and spin , and then we shall suffer for it : then the fox said , ' If you can do .'

You can perform such deeds as that blood , her husband , and that he had all that he had put this round him , and when he came to the bottom . said the huntsman washed his face all over with you , if you will see if it were .' ' Why did not lie down on the way , and now I will never get out of their beds . And as the girl went on the following day the wife , and never once stirred . ' But ,' said he to have , and that is passing in the kingdom must give him up , and two skulls , and carried away by the door to be just then there are all the good fairy told me he should do , when the snowflakes fell , and in it ; and she herself put on the summons ; and he answered ; ' there is a chaffinch ' s counsel , he said : ' The wind , so she said , ' is there ?'

```
    
- Grimms' Tales are strange and creepy no matter if it's machine or human writing it.
- At first, we started with a threshold of ```0.0```. This resulted in somewhat coherent sentences but did not quite flow. After increasing the threshold, the sentences greatly improved.
