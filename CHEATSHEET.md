bin/post -c cymbals cymbal_docs/makers.csv


# TO INDEX  A CSV FILE WITH ';' SEPAROTOR AND A HEADER LINE
curl 'http://localhost:8983/solr/cymbals/update?commit=true&header=true&separator=%3B' --data-binary @cymbal_docs/sabian.csv -H 'Content-type:application/csv'


# INDEX CSV 
java -Dtype=application/csv -Dc=rcymbals -jar example/exampledocs/post.jar example/cymbals/test_post.csv

#CREATE COLLECTION WITH CUSTOM CONFIG DIRECTORY
bin\solr create -c <collection_name> -d /<directory_name> -n <directory_name in zookeper> 

#TO REMOVE config directory for a collection from Zookeeper
#directory name is by default collection name
bin\solr zk rm -r /configs/<directory_name> -z localhost:9983

#TO COPY a folder from local to zookeeper
#!!! put SLASH AFTER DIRECTORY NAMES
bin\solr zk cp -r C:\\Users\\ygunduzalp\\Desktop\managed-schema  zk:configs/cymbals -z localhost:9983

EXAMPLE
bin/solr zk cp -r /home/yilmazgunalp/solr-7.2.1/mysyn.txt zk:/configs/cymbals/ -z localhost:9983

#remove managed-chema file from zookeeper
bin\solr zk rm /configs/cymbals/managed-schema -z localhost:9983

para
#TURN SCHEMALESS MODE OFF

 {"set-user-property": {"update.autoCreateFields":"false"}}
 
 
curl http://localhost:8983/solr/cymbals/config -H 'Content-type:application/json'  -d '{
  "update-requesthandler" : {
    "name": "/allocate",
    "class":"solr.SearchHandler",
    "defaults":{ "df":"brand" ,"rows":"5", "debug":"query", "fl":"id,brand,series,model,kind,size,score" }
    "useParams":"alloc"
    
  }
}'

##REPLACE FIELD TYPE

curl -X POST -H 'Content-type:application/json' --data-binary '{
  "replace-field-type":{
     
 "name":"model_shgls",
      "class":"solr.TextField",
      "positionIncrementGap":"100",
      "indexAnalyzer":{
        "tokenizer":{
          "class":"solr.StandardTokenizerFactory"},
        "filters":[{
            "class":"solr.StopFilterFactory",
            "words":"lang/stopwords_en.txt",
            "ignoreCase":"true"},
          {
            "class":"solr.LowerCaseFilterFactory"},
          {
            "class":"solr.EnglishPossessiveFilterFactory"},
          {
            "class":"solr.KeywordMarkerFilterFactory",
            "protected":"protwords.txt"},
		

            ]},
      "queryAnalyzer":{
        "tokenizer":{
          "class":"solr.StandardTokenizerFactory"},
        "filters":[{
            "class":"solr.SynonymGraphFilterFactory",
            "expand":"true",
            "ignoreCase":"true",
            "synonyms":"synonyms.txt"},
          {
            "class":"solr.StopFilterFactory",
            "words":"lang/stopwords_en.txt",
            "ignoreCase":"true"},
          {
            "class":"solr.EnglishPossessiveFilterFactory"},
          {
            "class":"solr.KeywordMarkerFilterFactory",
            "protected":"protwords.txt"},
         
          {
            "class":"solr.LowerCaseFilterFactory"}]}}


       }
}' http://localhost:8983/solr/cymbals/schema

##CREATE A REQUEST HANDLER

curl http://localhost:8983/solr/cymbals/config -H 'Content-type:application/json'  -d '{
  "update-requesthandler" : {
    "name": "/cymbalsets",
   "class":"solr.SearchHandler",
        "defaults":{
          "df":"brand",
          "rows":"5",
          "debug":"query",
          "fl":"id,brand,series,model,kind,size,score",
          "fq":"kind:set"

      }

  }
}'


##ADD FIELD TYPE


curl -X POST -H 'Content-type:application/json' --data-binary '{
  "replace-field-type" : {
     "name":"min_stemmed_text",
     "class":"solr.TextField",
     "positionIncrementGap":"100",
     "analyzer" : {
        "tokenizer":{
           "class":"solr.StandardTokenizerFactory" },
        "filters":[
        {"class":"solr.LowerCaseFilterFactory"},
		{
            "class":"solr.StopFilterFactory",
            "words":"lang/stopwords_en.txt",
            "ignoreCase":"true"},
        {"class":"solr.EnglishMinimalStemFilterFactory"}

            ]}}
}' http://localhost:8983/solr/cymbals/schema


##REPLACE A FIELD

curl -X POST -H 'Content-type:application/json' --data-binary '{
  "replace-field":{
     "name":"series",
     "type":"min_stemmed_text"
     }
}' http://localhost:8983/solr/cymbals/schema

##synoym FILED TYPE

curl -X POST -H 'Content-type:application/json' --data-binary '{
  "replace-field-type": {
     "name":"min_stemmed_text",
     "class":"solr.TextField",
     "positionIncrementGap":"100",
     "indexAnalyzer": {
        "tokenizer": {"class":"solr.StandardTokenizerFactory" },
        "filters": [
        {"class":"solr.LowerCaseFilterFactory"},
		{"class":"solr.StopFilterFactory",
            "words":"lang/stopwords_en.txt",
            "ignoreCase":"true"},
        {"class":"solr.EnglishMinimalStemFilterFactory"}]},
      "queryAnalyzer": {
        "tokenizer": {"class":"solr.StandardTokenizerFactory" },
        "filters":[
        {"class":"solr.SynonymGraphFilterFactory",
            "expand":"true",
            "ignoreCase":"true",
            "synonyms":"synonyms.txt"},

        {"class":"solr.LowerCaseFilterFactory"},
		{"class":"solr.StopFilterFactory",
            "words":"lang/stopwords_en.txt",
            "ignoreCase":"true"},
        {"class":"solr.EnglishMinimalStemFilterFactory"}]}      
  }          
}' http://localhost:8983/solr/cymbals/schema



curl -X POST -H 'Content-type:application/json' --data-binary '{
  "replace-field-type" : {
     "name":"synonym_text",
     "class":"solr.TextField",
     "positionIncrementGap":"100",
     "indexanalyzer" : {
        "tokenizer":{
           "class":"solr.StandardTokenizerFactory" },
        "filters":[
        {"class":"solr.LowerCaseFilterFactory"},
		{
            "class":"solr.StopFilterFactory",
            "words":"lang/stopwords_en.txt",
            "ignoreCase":"true"},
        {"class":"solr.EnglishMinimalStemFilterFactory"}]},
      "queryanalyzer" : {
        "tokenizer":{
           "class":"solr.StandardTokenizerFactory" },
        "filters":[
        {
            "class":"solr.SynonymGraphFilterFactory",
            "expand":"true",
            "ignoreCase":"true",
            "synonyms":"mysyn.txt"},

        {"class":"solr.LowerCaseFilterFactory"},
		{
            "class":"solr.StopFilterFactory",
            "words":"lang/stopwords_en.txt",
            "ignoreCase":"true"},
        {"class":"solr.EnglishMinimalStemFilterFactory"}]}      
           
}' http://localhost:8983/solr/cymbals/schema


 curl http://localhost:8983/solr/cymbals/config/params -H 'Content-type:application/json'  -d '{
  "update":{
    "alloc":{
      "defType":"edismax",
      "df":"brand",
      "rows":5,
      "sow":"true",
      "fl": "id,brand,series,model,kind,size,score",
      "fq": ["{!df=size v=$q}","{!df=kind v=$q}","{!df=series v=$q}"]},
      "bf": "query()"
	  }}'




      curl http://localhost:8983/solr/cymbals/config/params -H 'Content-type:application/json'  -d '{
  "update":{
    "alloc":{
      "defType":"edismax",
      "df":"brand",
      "rows":5,
      "sow":"true",
      "fl": "id,brand,series,model,kind,size,score",
      "fq": ["{!df=size v=$q}","{!df=kind v=$q}","{!df=series v=$q}"]}
      	
      }}'



curl http://localhost:8983/solr/cymbals/config/params -H 'Content-type:application/json'  -d '{
  "set":{
    "cymbalsets":{
      "defType":"edismax",
      "df":"brand",
      "rows":5,
      "sow":"true",
      "fl": "id,brand,series,model,kind,size,score",
      "fq": ["kind:set","{!df=series v=$q}"]}
      	
      }}'



curl -X POST http://localhost:3000/makers/3233/?r=9280




curl http://localhost:8983/solr/cymbals/config/params -H 'Content-type:application/json'  -d '{
   "update":{
    "find":{
      "defType":"edismax",
        "df":"catch_all",
        "sow":"true",
        "fl":"id,brand,series,model,kind,size,score",
        "fq": ["{!collapse field=brand}"],
        "expand": "true"}
  }
}'















