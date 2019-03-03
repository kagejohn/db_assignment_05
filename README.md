# Assignment 5

## Excercise 1

```mysql
CREATE DEFINER=`root`@`%` PROCEDURE `denormalizeComments`(IN wherePostID int)
BEGIN
	SET @result := 
		(select 
			concat('[',
				GROUP_CONCAT(
					JSON_OBJECT(    
						'Id:', Id, 
						'PostId:', PostId, 
						'Score:', Score, 
						'Text:', Text, 
						'CreationDate:', CreationDate, 
						'UserId:', UserId
					)
				SEPARATOR ',')
			,']')
		from comments where PostId = wherePostID);
	UPDATE `stackoverflow`.`posts`
		SET
		`CommentJsonArray` = @result
		WHERE `Id` = wherePostID;
END
```

## Exercise 2

```mysql
CREATE TRIGGER newCommentAdded 
after insert ON comments 
for each row

update posts
	set  CommentJsonArray=JSON_ARRAY_APPEND(CommentJsonArray, '$', 
		JSON_OBJECT(
				'Id:', NEW.Id,
				'Text:', NEW.Text,
				'Score:', NEW.Score,
				'PostId:', NEW.PostId,
				'UserId:', NEW.UserId,
				'CreationDate:', NEW.CreationDate))
	where Id = NEW.PostId;
```

## Exercise 3

```mysql
CREATE DEFINER=`root`@`%` PROCEDURE `createComment`(IN Id int, PostId int, Score int, Text text, CreationDate datetime, UserId int)
BEGIN
	INSERT INTO `stackoverflow`.`comments`
		(`Id`, `PostId`, `Score`, `Text`, `CreationDate`, `UserId`)
		VALUES
		(Id, PostId, Score, Text, CreationDate, UserId);
	update posts
	set  CommentJsonArray=JSON_ARRAY_APPEND(CommentJsonArray, '$', 
		JSON_OBJECT(
				'Id:', Id,
				'Text:', Text,
				'Score:', Score,
				'PostId:', PostId,
				'UserId:', UserId,
				'CreationDate:', CreationDate))
	where Id = PostId;
END
```

## Exercise 4

```mysql
CREATE DEFINER=`root`@`%` PROCEDURE `refreshMvQuestions`()
BEGIN
	declare questionCount int default 0;
	declare i int default 0;
	declare j int default 0;
    declare jsonQuestion JSON;
    declare jsonAnswer JSON;
    
    TRUNCATE TABLE mv_questions;
    
	select Count(*) from posts where PostTypeId = 1 into questionCount;
    
    set i = 0;
    while i < questionCount do
		set jsonAnswer = JSON_ARRAY();
        set jsonQuestion = JSON_ARRAY();
    
		select 
			posts.Id, Body as Question, Score, users.DisplayName, AnswerCount 
		into
			@PostsId, @Question, @ScoreQ, @DisplayNameQ, @AnswerCount
		from posts left join  users on posts.OwnerUserId = users.Id where PostTypeId = 1 limit i, 1; -- select one question
    
		set j = 0;
		while j < @AnswerCount do
        
			select 
				Body as Answer, Score, users.DisplayName 
			into
				@Answer, @ScoreA, @DisplayNameA
			from posts left join  users on posts.OwnerUserId = users.Id where posts.Id = i+1 limit j, 1; -- select one answer for the current question
            
            set jsonAnswer = JSON_ARRAY_APPEND(jsonAnswer, '$', 
				JSON_OBJECT(
						'Answer:', @Answer,
						'Score:', @ScoreA,
						'DisplayName:', @DisplayNameA));
                        
		set j = j + 1;
		end while; -- Answer
        
        set jsonQuestion = JSON_ARRAY_APPEND(jsonQuestion, '$', 
				JSON_OBJECT(
						'Question:', @Question,
						'Score:', @ScoreQ,
						'DisplayName:', @DisplayNameQ,
                        'Answers:', jsonAnswer));
                        
		INSERT INTO mv_questions (`question`) VALUES (jsonQuestion);
    
    set i = i + 1;
    end while; -- Question
END
```

## Exercise 5

```mysql
TODO
```
