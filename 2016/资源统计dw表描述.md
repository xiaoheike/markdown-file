## edu_video_stat_daily ##
教材->章节->视频数量统计
```
1. sum_date					varchar(8)			统计日期		"20160427"
2. subject					varchar(255)		学科			"$SB0300" 英语
3. edition					varchar(255)		版本			"$E001000" 人教版
4. grade					varchar(255)		年级			"$ON030300" 九年级
5. phase					varchar(255)		年段			"$ON030000" 初中
6. sub_edition				varchar(255)		子版本		"$E001001" 人教版下册
7. teachingmaterials		varchar(255)		教材唯一id
8. teachingmaterials_title	varchar(1024)		描述			"河北教育出版社初中英语九年级全一册（衔接三年级起点）（2014版）"
9. chapters					varchar(1024)		章节唯一id	
10. video_count				int					该教材章节下视频总数
11. video_audited_count		int					该教材章节下视频审核数量
12. insert_time				datetime			刷新统计时间	"2016-02-02 10:14:03"
```
## edu_audio_stat_daily ##
教材->章节->音频数量统计
```
1. sum_date					varchar(8)			统计日期
2. subject					varchar(255)		学科
3. edition					varchar(255)		版本
4. grade					varchar(255)		年级
5. phase					varchar(255)		年段
6. sub_edition				varchar(255)		子版本
7. teachingmaterials		varchar(255)		教材唯一id
8. teachingmaterials_title	varchar(1024)		教材描述
9. chapters					varchar(1024)		章节唯一id
10. audio_count				int					该教材章节下音频数量
11. audio_audited_count		int					该教材章节下图片审核数量
12. insert_time				datetime			刷新统计时间
```
## edu_image_stat_daily ##
教材->章节->图片数量统计
```
1. sum_date					varchar(8)			统计日期
2. subject					varchar(255)		学科
3. edition					varchar(255)		版本
4. grade					varchar(255)		年级
5. phase					varchar(255)		年段
6. sub_edition				varchar(255)		子版本
7. provider					varchar(1024)		提供商
8. teachingmaterials		varchar(255)		教材唯一id
9. teachingmaterials_title	varchar(1024)		教材描述
10. chapters				varchar(1024)		章节唯一id
11. image_count				int					该教材章节下图片数量
12. image_audited_count		int					该教材章节下图片审核数量
13. insert_time				datetime			刷新统计时间
```
### edu_questions_stat_daily ###
教材->章节->习题数量统计('questions', 'coursewareobjects')
```
1. sum_date					varchar(8)			统计日期
2. subject					varchar(255)		学科
3. edition					varchar(255)		版本
4. grade					varchar(255)		年级
5. phase					varchar(255)		年段
6. sub_edition				varchar(255)		子版本
7. questionsource			varchar(255)		习题唯一id
8. provider					varchar(1024)		提供商
9. teachingmaterials		varchar(255)		教材唯一id
10. teachingmaterials_title	varchar(1024)		教材描述
11. chapters				varchar(1024)		章节唯一id
12. questions_count			int					该教材章节下习题数量
13. questions_audited_count	int					该教材章节下习题审核数量
14. insert_time				datetime			刷新统计时间
```
## edu_coursewares_stat_daily ##
教材->章节->课件数量统计
```
1. sum_date					varchar(8)			统计时间
2. subject					varchar(255)		学科
3. edition					varchar(255)		版本
4. grade					varchar(255)		年级
5. phase					varchar(255)		年段
6. sub_edition				varchar(255)		子版本
7. teachingmaterials		varchar(255)		教材唯一id
8. teachingmaterials_title	varchar(1024)		教材描述
9. chapters					varchar(1024)		章节唯一id
10. coursewares_count			int				该教材章节下课件数量
11. coursewares_audited_count	int				该教材章节下课件审核数量
12. insert_time				datetime			刷新统计时间
```
## edu_resource_stat_daily ## 
每日资源统计，包括习题、学案、音频等等
```
1. sum_date             	varchar(8)			统计时间
2. resource_code        	varchar(255)		nd自定义编码 $RA0201（教材）
3. resource_type        	varchar(255)		资源类型 teachingmaterials（教材）
4. resource_name        	varchar(1024)		资源名称 教材
5. source_count         	bigint(20)   		资源总数
6. audited_count        	bigint(20)   		审计总数
7. new_increment_count  	bigint(20)   		新增总数（）
8. net_increment_count  	bigint(20)   		净增总数（减去可能删除的部分）
9. insert_time          	datetime  			刷新时间
```



## 资源统计修改 ##
1. question，coursewareobject -> question库
2. ndreasource中关于question以及coursewareobject部分数据 -> question库
证明语句：
	select count(*) FROM questions a INNER JOIN ndresource b ON a.identifier = b.identifier;
上述语句返回值与questions数据数量相同，说明questions表的关系都被放置到question库中
3. res_coverage中关于question以及coursewareobject部分数据 -> question库
证明语句：
	SELECT count(*) from res_coverages a INNER JOIN esp_resource_detail_log_question.questions b ON a.resource = b.identifier
	SELECT count(*) from res_coverages a INNER JOIN questions b ON a.resource = b.identifier
第一条语句返回值为零，第二条语句返回值与questions表总数想接近。
4. resource_categories中关于question以及coursewareobject部分数据 -> question库  res_type
证明语句：
	SELECT count(*) from resource_categories a INNER JOIN esp_resource_detail_log_question.questions b ON a.resource = b.identifier
	SELECT count(*) from resource_categories a INNER JOIN questions b ON a.resource = b.identifier
第一条语句返回值为零，第二条语句返回值不为零且数据较大
5. resource_relation，从震宇获得信息
“章节”--“习题” 旧库
“习题”--“课件” 新库
其他的关系都在旧库
6. lc分库分表没有影响的统计功能点
- nd org 年段、年级、教材、章节下的音频统计
- nd org 年段、年级、教材、章节下的视频统计
- nd org 年段、年级、教材、章节下的图片统计
- nd org 年段、年级、教材、章节下的学案统计
- nd org 年段、年级、教材、章节节数，小结束统计
- nd org 年段、年级、教材、章节下的教案数量
- nd org 年段、年级、教材、章节下的电子教材(ebook)数量
- nd org 年段、年级、教材、章节下的学案统计
- 
验证语句（习题分库不受影响统计）：
```SQL
	SELECT * from edu_audio_stat_daily WHERE sum_date = "20160502" limit 0, 10
	SELECT * from edu_chapter_stat_daily WHERE sum_date = "20160502" limit 0, 10
	SELECT * from edu_coursewares_stat_daily WHERE sum_date = "20160502" limit 0, 10
	SELECT * from edu_ebooks_stat_daily WHERE sum_date = "20160502" limit 0, 10
	SELECT * from edu_image_stat_daily WHERE sum_date = "20160502" limit 0, 10
	SELECT * from edu_learningplan_stat_daily WHERE sum_date = "20160502" limit 0, 10
	SELECT * from edu_lessonplan_stat_daily WHERE sum_date = "20160502" limit 0, 10
	SELECT * from edu_resource_stat_week WHERE sum_date = "20160501" limit 0, 10
	SELECT * from edu_video_stat_daily WHERE sum_date = "20160502" limit 0, 10
```
7. lc分库分表影响的统计功能点
- nd org 年段、年级、教材、章节下的习题统计
- 日总量统计

验证语句（习题分库影响统计）：
```SQL
	SELECT * from edu_questions_stat_daily WHERE sum_date = "20160502" limit 0, 10
	SELECT * from edu_resource_stat_daily WHERE sum_date = "20160502" limit 0, 10
```