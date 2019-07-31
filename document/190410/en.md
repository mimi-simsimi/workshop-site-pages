<style
  type="text/css">
style {color:#ffffff;display:hidden}
h1, h2, h3, h4, h5, h6 {color:#333333;}
p, li {color:#333333}
code {color:#000080;}
</style>

# SmallTalk API
Make your chatbot users laugh. With over 100 million chatbot-dedicated expressions created by tens of millions of people around the world and a conversation engine(AICR) that has supported the world's best chatbot for over 15 years, you'll have brilliant chatting skills in your chatbot. You can take advantage of most of the expressions in 'SimSimi', the best chatbot for small talk.

## Concept
A ‘talkset’ consists of a pair of question-answer sentences(`qtext`-`atext`). SWS has more than 100 million talkset repository.

<img src="https://workshop.simsimi.com/static/img/smalltalk_diagram_01e.png" width="300px" alt="talkset concept">

When a request is received through the SmallTalk API, SWS’s conversation engine(AICR) finds the relevant question sentences(`qtext`) in the talkset repository considering the similarities with a user’s request sentence(`utext`) and some other factors, creates candidate talksets, and selects the most appropriate talkset filtering/weighing the parameters included in the request and other conditions.

The answer sentence provided by the SmallTalk API is the atext of the talkset selected through this process. If a requested `utext` is “Have you eaten lunch?”, the SmallTalk API returns an `atext` through the process like below.

<img src="https://workshop.simsimi.com/static/img/smalltalk_diagram_02e.png" width="600px" alt="smalltalk API flow">

## Basic Request
You can receive a response by requesting to the SmallTalk API endpoint(`https://wsapi.simsimi.com/{VERSION}/talk`) with the appropriate method(`POST`), project key, and required parameters(`utext`, `lang`).

#### Example Request
``` bash
curl -X POST https://wsapi.simsimi.com/190410/talk \
     -H "Content-Type: application/json" \
     -H "x-api-key: PASTE_YOUR_PROJECT_KEY_HERE" \
     -d '{
            "utext": "hello", 
            "lang": "en" 
     }'                     
```
- `utext` : Sentence that a user entered in your chatbot. (100 characters max)
- `lang` : Language code ([Language-language code table](#list-of-supported-languages-and-language-codes))

#### Example Response
``` json
{
  "status":200,
  "statusMessage":"OK",
  "atext":"Hello it is a pleasure to meet you!",
  "lang":"en",
  "request":{
    "utext":"hello",
    "lang":"en"
    }
}    
```
- `atext` : answer sentence
- `request` : request body
- `status`, `statusMessage` : status information ([status code list](#status-code-and-status-messages))

## Response Control
SmallTalk API provides options for controlling response. 

#### Example Request
If you want your bot to answer sentences(`atext`) with a badword probability of 70% or less among the talksets generated in the United States or Canada, you can request by adding the following two options `country` and `atext_bad_prob_max`.
``` bash
curl -X POST https://wsapi.simsimi.com/190410/talk \
     -H "Content-Type: application/json" \
     -H "x-api-key: PASTE_YOUR_PROJECT_KEY_HERE" \
     -d '{
            "utext":"hello",
            "lang": "en",
            "country" : ["US", "CA"],
            "atext_bad_prob_max": 0.7
     }'  
 ```

## Response Control Options
The options for controlling the response in the current version of the SmallTalk API are:

- `country` 
  + Specifies the countries of the talkset. You can specify this value when you are requesting SmallTalk API with a language that is used in multiple countries to filter the talksets created in the desired countries.
  + The format is an array of [ISO-3166-1 alpha-2](https://en.wikipedia.org/wiki/ISO_3166-1_alpha-2#Officially_assigned_code_elements) country codes (up to 10).
　
- `atext_bad_prob_max`, `qtext_bad_prob_max`, `talkset_bad_prob_max`
  + Limits the maximum of badword probability in the sentences. 
  + It's used to suppress bad words in chatbot answers. In many cases, it is sufficient to adjust the bad word probability maximum of the answer sentence(`atext`) appropriately, and  you can further control the question sentence(`qtext`) and the talkset(`talkset`) by specifying an additional bad word probability. The bad probability of a talkset(`talkset`) is determined by combining the question and answer sentences into one sentence. 
  + Specifies one of the values from `0.0` to `1.0` with the first decimal place. Defaults to `1.0`.
　
- `atext_bad_prob_min`
  + Limits the minimum of badword probability in the answer sentence(`atext`). 
  + This is an option that you can use to make chatbots say bad words. Many platforms have limitations related to the integrity of the content, so please be aware of their use.
  + Specifies one of the values from `0.0` to `1.0` with the first decimal place. It must be equal to or less than the `atext_bad_prob_max`. Defaults to `1.0`. 
　
- `atext_length_max`, `atext_length_min`
  + Specify the length of the answer sentence(`atext`).
  + You can set the length depending on the chatbot's personality or conversation situation. 
  + The `atext` length is from `1` to `256` characters. Defaults to `1`.
　
- `regist_date_max`, `regist_date_min`
  + Specify the range of the date on which the `talkset` was created.
  + It can be used to implement the latest trend sensitive chatbots and chatbots that remain in the past. 
  + The default format is `yyyy-MM-dd HH:mm:ss`. If not specified, `regist_date_max` defaults to the current time,` regist_date_min` is the first talkset creation date.

## Request Additional Information
The SmallTalk API provides a way to get more detailed information about responses. Define `cf_info` in the request `body` for the additional information you want to receive.

#### Example Request
``` bash
curl -X POST https://wsapi.simsimi.com/190410/talk \
     -H "Content-Type: application/json" \
     -H "x-api-key: PASTE_YOUR_PROJECT_KEY_HERE" \
     -d '{
            "utext":"hello",
            "lang": "en",
            "cf_info" : [
                  "qtext",
                  "country",
                  "atext_bad_prob",
                  "atext_bad_type",
                  "regist_date"
            ],
     }'         
```
#### Example Response
``` json
{
    "status" : 200,
    "statusMessage" : "OK",
    "atext" : "Hello it is a pleasure to meet you!",
    "lang" : "en",
    "utext" : "hello",
    "qtext" : "hello~~",
    "country" : "US",
    "atext_bad_prob" : 0.0,
    "atext_bad_type" : "dpd",
    "regist_date" : "2017-07-08 08:24:37"
}                           
  
```

## Additional Information Details
- `qtext`
  + The question sentence(`qtext`) that is a pair of the answer sentence(`atext`) in the talkset.
- `country`
  + `ISO-3166-1 alpha-2` country code for the talkset. (See the [ISO-3166-1 alpha-2](https://en.wikipedia.org/wiki/ISO_3166-1_alpha-2#Officially_assigned_code_elements))
- `atext_bad_prob`
  + The badword probability of the answer sentence(`atext`).
- `atext_bad_type`
  + The distinguishing technique used to estimate badword probability for the answer sentence(`atext`). It is one of `STAPX`, `DPD`, `WPF` and `HB10A` (See the [Badword Classification Techniques (Korean)](http://blog.simsimi.com/2019/03/blog-post.html) )
- `regist_date`
  + The creation time of the talkset.
 
## List of supported languages and language codes
Most language codes are the same as ISO-639-1, but note that there are other cases(*).

|Language Name	|	 Name(Native)	|	Language Code|
| --- | --- | --- |
|Afrikaans	|	 Afrikaans	|	 af|
|Albanian	|	 Shqip	|	 al*|
|Arabic	|	 العربية	|	 ar|
|Armenian	|	 հայերեն	|	 hy|
|Azerbaijani	|	 Azərbaycanca	|	 az|
|Basque	|	 euskara	|	 eu|
|Belarusian	|	 беларуская	|	 be|
|Bengali	|	 বাংলা	|	 bn|
|Bosnian	|	 bosanski	|	 bs|
|Bulgarian	|	 български	|	 bg|
|Catalan	|	 català	|	 ca|
|Cebuano	|	 Cebuano	|	 cx*|
|Chinese	|	 中文	|	 ch*|
|Croatian	|	 hrvatski	|	 hr|
|Czech	|	 čeština	|	 cs|
|Danish	|	 Dansk	|	 da|
|Dutch	|	 Nederlands	|	 nl|
|English	|	 English	|	 en|
|Estonian	|	 eesti	|	 et|
|Filipino	|	 Filipino	|	 ph*|
|Finnish	|	 suomi	|	 fi|
|French	|	 Français	|	 fr|
|Frisian	|	 Frysk	|	 fy|
|Galician	|	 galego	|	 gl|
|Georgian	|	 ქართული	|	 ka|
|German	|	 Deutsch	|	 de|
|Greek	|	 Ελληνικά	|	 el|
|Gujarati	|	 ગુજરાતી	|	 gu|
|Hebrew	|	 עברית	|	 he|
|Hindi	|	 हिन्दी	|	 hi|
|Hungarian	|	 magyar	|	 hu|
|Icelandic	|	 íslenska	|	 is|
|Indonesian	|	 Bahasa Indonesia	|	 id|
|Italian	|	 Italiano	|	 it|
|Japanese	|	 日本語	|	 ja|
|Kannada	|	 ಕನ್ನಡ	|	 kn|
|Kazakh	|	 қазақ	|	 kk|
|Khmer	|	 ភាសាខ្មែរ	|	 kh*|
|Korean	|	 한국어	|	 ko|
|Kurdish (Kurmanji)	|	 Kurdî (Kurmancî)	|	 ku|
|Latvian	|	 latviešu	|	 lv|
|Lithuanian	|	 lietuvių	|	 lt|
|Macedonian	|	 македонски	|	 mk|
|Malay	|	 Melayu	|	 ms|
|Malayalam	|	 മലയാളം	|	 ml|
|Marathi	|	 मराठी	|	 mr|
|Mongolian	|	 Монгол	|	 mn|
|Myanmar (Burmese)	|	 မြန်မာဘာသာ	|	 my|
|Nepali	|	 नेपाली	|	 ne|
|Norwegian	|	 norsk	|	 nb|
|Assamese	|	 অসমীয়া	|	 as|
|Breton	|	 Brezhoneg	|	 br|
|Guaraní	|	 Guarani	|	 gn|
|Javanese	|	 Basa Jawa	|	 jv|
|Oriya	|	 ଓଡ଼ିଆ	|	 or|
|Kinyarwanda	|	 Ikinyarwanda	|	 rw|
|Chinese – Traditional	|	 繁體中文	|	 zh*|
|Pashto	|	 پښتو	|	 ps|
|Persian	|	 فارسی	|	 fa|
|Polish	|	 polski	|	 pl|
|Portuguese	|	 português	|	 pt|
|Punjabi	|	 ਪੰਜਾਬੀ	|	 pa|
|Romanian	|	 Română	|	 ro|
|Russian	|	 русский	|	 ru|
|Serbian	|	 српски	|	 rs*|
|Sinhala	|	 සිංහල	|	 si|
|Slovak	|	 slovenčina	|	 sk|
|Slovenian	|	 slovenščina	|	 sl|
|Spanish	|	 español	|	 es|
|Swahili	|	 Kiswahili	|	 sw|
|Swedish	|	 svenska	|	 sv|
|Tajik	|	 Тоҷикӣ	|	 tg|
|Tamil	|	 தமிழ்	|	 ta|
|Telugu	|	 తెలుగు	|	 te|
|Thai	|	 ภาษาไทย	|	 th|
|Turkish	|	 Türkçe	|	 tr|
|Ukrainian	|	 українська	|	 uk|
|Urdu	|	 Урду	|	 ur|
|Uzbek	|	 O‘zbek	|	 uz|
|Vietnamese	|	 Tiếng Việt	|	 vn*|
|Welsh	|	 Cymraeg	|	 cy|

## Status Code and Status Messages

|`status` | `statusMessage` | Description | 
| --- | --- | --- |
|200 | 	OK | Everything worked as expected. |
|227 | 	Parameter Required | The request is missing a required parameter. |
|228 |	Do Not Understand | There is no proper answer to the question you asked. |
|403 |	Unauthorized | Invalid API Key. |
|429 |	Limit Exceeded | The user has exceeded the quota. |
|500 |	Server error | Something went wrong on SWS's end. |

## Badword Probability
The badword probability is an index developed by the SimSimi team to identify how unhealthy/malicious a sentence is, and there are some distinguishing techniques for index calculation including advanced deep learning with superior performance. For more information, please see the blog post, [Malcious Sentence Classification Techniques in the SimSimi Service (Korean)](http://blog.simsimi.com/2019/03/blog-post.html).
