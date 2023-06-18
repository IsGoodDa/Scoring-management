### 为了实现该功能，我们可以使用 Java 编程语言编写代码来连接数据库，并定义一个 Score 实体类用于存储评分信息和提供相应的操作方法，如添加评分、查询所有评分等。具体的代码实现可以参考以下示例；

## To achieve this functionality, we can write code in the Java programming language to connect to the database and define a Score entity class to store scoring information and provide corresponding action methods, such as adding scores, querying all scores, and so on. For specific code implementation, refer to the following example.

### 以下是评分管理模块的Java代码示例：

## Here is a Java code example for the Score Management module:


package com.ruoyi.kx.controller;

import com.ruoyi.common.core.controller.BaseController;
import com.ruoyi.common.core.domain.AjaxResult;
import com.ruoyi.common.core.page.TableDataInfo;
import com.ruoyi.kx.bean.ScoreSort;
import com.ruoyi.kx.domain.*;
import com.ruoyi.kx.mapper.*;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.*;

import java.math.BigDecimal;
import java.math.RoundingMode;
import java.text.DecimalFormat;
import java.util.ArrayList;
import java.util.List;

@Controller
@RequestMapping("/kx/data")
public class DataAnalysisController extends BaseController {


    @Autowired
    KxMarkMapper kxMarkMapper;
    @Autowired
    KxStudentMapper kxStudentMapper;
    @Autowired
    KxTimingMapper kxTimingMapper;
    @Autowired
    KxScoreRecordMapper kxScoreRecordMapper;
    @Autowired
    KxRateOftenMapper kxRateOftenMapper;
    @Autowired
    KxClassesMapper kxClassesMapper;
    @Autowired
    KxTimingRateMapper kxTimingRateMapper;
    @Autowired
    KxTeacherMapper kxTeacherMapper;

    @CrossOrigin
    @ResponseBody
    @GetMapping("/student/{classId}")
    public TableDataInfo student(@PathVariable("classId") Long classId) {
        List<KxStudent> list = kxStudentMapper.selectKxStudentByClassId(classId);
        return getDataTable(list);
    }

    @CrossOrigin
    @ResponseBody
    @GetMapping("/partRate/{classId}/{timingRateId}")
    public AjaxResult partRate(@PathVariable Long classId, @PathVariable Long timingRateId) {
        List<KxStudent> kxStudents = kxStudentMapper.selectKxStudentByClassId(classId);

        if (kxStudents == null || kxStudents.size() == 0) {
            return error("该班级不存在");
        }
        KxTimingRate kxTimingRate = kxTimingRateMapper.selectKxTimingRateById(timingRateId);
        if (kxTimingRate == null) {
            return error("周期评不存在");
        }

        Long count = kxTimingRateMapper.selectPeriodCount(kxTimingRate.getType());

        long l = count * kxStudents.size();

        List<KxScoreRecord> kxScoreRecords = kxScoreRecordMapper.selectKxScoreRecordByClassIdAndTimingRateId(classId, timingRateId);
        float l1 = (float) kxScoreRecords.size() / l;
        return success().put("data",l1);
    }

    /*查看学生发展评价分数*/
    @CrossOrigin
    @ResponseBody
    @GetMapping("/select/fiveRate/{studentId}/{timingId}")
    public AjaxResult rateFiveScore(@PathVariable Long studentId, @PathVariable Long timingId) {
        KxTiming kxTiming = kxTimingMapper.selectKxTimingById(timingId);
        KxStudent kxStudent = kxStudentMapper.selectKxStudentById(studentId);
        if (kxTiming == null) {
            return error("找不到该学期");
        }

        if (kxStudent == null) {
            return error("找不到该学生");
        }
        FiveRate fiveRate = new FiveRate(getScoreSort("A", kxTiming, kxStudent).getScore(), getScoreSort("B", kxTiming, kxStudent).getScore(), getScoreSort("C", kxTiming, kxStudent).getScore(), getScoreSort("D", kxTiming, kxStudent).getScore(), getScoreSort("E", kxTiming, kxStudent).getScore());

        return success().put("data", fiveRate);

    }

    private ScoreSort getScoreSort(String type, KxTiming kxTiming, KxStudent kxStudent) {
        ScoreSort scoreSort = kxScoreRecordMapper.selectKxScoreRecordBySort(kxStudent.getId(), type, 1L, kxTiming.getId());
        DecimalFormat df = new DecimalFormat("#.##"); // 创建 DecimalFormat 对象，指定格式为保留最多两位小数
        df.setRoundingMode(RoundingMode.DOWN); // 设置舍入模式为向下舍入
        float rounded;


        {
            float v = kxRateOftenMapper.selectKxScoreOftenAvRate(kxStudent.getId(), kxTiming.getId());
            rounded = Float.parseFloat(df.format(scoreSort.getScore() + v)); // 格式化小数部分并将结果转换为 float 类型
        }

        String result = df.format(rounded);
        scoreSort.setStudentName(kxStudent.getName());
        scoreSort.setImagePath(kxStudent.getImagePath());
        scoreSort.setScore(Float.parseFloat(result));
        KxMark kxMark = kxStudent.getKxMark();

        if (kxMark != null) {
            char firstChar = kxStudent.getClasses().getName().charAt(0);
            if (firstChar == '一' || firstChar == '二') {

                scoreSort.setScore(changeFloat(scoreSort.getScore() + getRateMarkScore(kxMark.getMoralScore(), true).floatValue() + getRateMarkScore(kxMark.getChineseScore(), true).floatValue() + getRateMarkScore(kxMark.getMathScore(), true).floatValue() + getRateMarkScore(kxMark.getScienceScore(), true).floatValue() + getRateMarkScore(kxMark.getPhysicalScore(), true).floatValue() + getRateMarkScore(kxMark.getArtScore(), true).floatValue() + getRateMarkScore(kxMark.getMusicScore(), true).floatValue() + getRateMarkScore(kxMark.getLaoDongScore(), true).floatValue()
//                                       getRateMarkScore(kxMark.getEnglishScore(), true).floatValue()
                ));


            } else {
                logger.info("kxMark111:{}", kxMark.getMoralScore());

                scoreSort.setScore(changeFloat(scoreSort.getScore() + getRateMarkScore(kxMark.getMoralScore(), false).floatValue() + getRateMarkScore(kxMark.getChineseScore(), false).floatValue() + getRateMarkScore(kxMark.getMathScore(), false).floatValue() + getRateMarkScore(kxMark.getScienceScore(), false).floatValue() + getRateMarkScore(kxMark.getPhysicalScore(), false).floatValue() + getRateMarkScore(kxMark.getArtScore(), false).floatValue() + getRateMarkScore(kxMark.getMusicScore(), false).floatValue() + getRateMarkScore(kxMark.getEnglishScore(), false).floatValue()));


            }

        }
        return scoreSort;
    }

    /*查询男女比例*/
    @CrossOrigin
    @GetMapping("/select/ratioOfMenToWomen")
    @ResponseBody
    public AjaxResult ratioOfMenToWomen() {
        List<RatioOfMenToWomen> ratioOfMenToWomen = kxStudentMapper.ratioOfMenToWomen();
        if (ratioOfMenToWomen == null || ratioOfMenToWomen.size() == 0) {
            return error("未查询到结果");
        } else {
            return success().put("data", ratioOfMenToWomen);
        }
    }

    @CrossOrigin
    @GetMapping("/select/selectTeacherNum")
    @ResponseBody
    public AjaxResult selectTeacherNum() {
        Long aLong = kxTeacherMapper.selectKxTeacherNumber();
        if (aLong == null) {
            return error("未查询到结果");
        } else {
            return success().put("data", aLong);
        }
    }

    @CrossOrigin
    @GetMapping("/select/numberOfReviews")
    @ResponseBody
    public AjaxResult numberOfReviews() {
        NumberOfReviews numberOfReviews = kxScoreRecordMapper.NumberOfReviews();
        if (numberOfReviews == null) {
            return error("未查询到结果");
        } else {
            return success().put("data", numberOfReviews);
        }
    }

    @CrossOrigin
    @GetMapping("/select/week")
    @ResponseBody
    public AjaxResult queryWeek() {

        List<KxTimingRate> kxTimingRates = kxTimingRateMapper.queryWeek();
        if (kxTimingRates.size() == 0) {
            return error("未找到结果");
        } else {
            return success().put("last", kxTimingRates.get(0)).put("now", kxTimingRates.get(1));
        }
    }


    @CrossOrigin
    @GetMapping("/select/month")
    @ResponseBody
    public AjaxResult queryMonth() {

        List<KxTimingRate> kxTimingRates = kxTimingRateMapper.queryMonth();
        if (kxTimingRates.size() == 0) {
            return error("未找到结果");
        } else {
            return success().put("last", kxTimingRates.get(0)).put("now", kxTimingRates.get(1));
        }
    }


    @CrossOrigin
    @GetMapping("/class/avMark/{classId}/{timingId}")
    @ResponseBody
    public AjaxResult ClassSelectKxMarkAv(@PathVariable Long classId, @PathVariable Long timingId) {
        AvMark kxMark = kxMarkMapper.selectKxMarkAv(classId, timingId);
        if (kxMark == null) {
            return error("未查询到结果");
        } else {
            return success().put("data", kxMark);
        }
    }

    /*查询年级平均分*/
    @CrossOrigin
    @GetMapping("/grande/avMark/{name}/{timingId}")
    @ResponseBody
    public AjaxResult grandeSelectKxMarkAv(@PathVariable String name, @PathVariable Long timingId) {
        AvMark kxMark = kxMarkMapper.grandeSelectKxMarkAv(name, timingId);
        if (kxMark == null) {
            return error("未查询到结果");
        } else {
            return success().put("class", name + "年级").put("rows", kxMark);
        }
    }

    public Float changeFloat(Float number) {
        DecimalFormat decimalFormat = new DecimalFormat("#.#");
        String formattedNumber = decimalFormat.format(number);
        return Float.parseFloat(formattedNumber);
    }

    public BigDecimal getRateMarkScore(BigDecimal score, boolean flag) {

        if (flag) {
            return score;
        }
        if (score != null) {
            if (score.compareTo(BigDecimal.valueOf(90)) >= 0) {
                return BigDecimal.valueOf(5);
            } else if (score.compareTo(BigDecimal.valueOf(80)) >= 0) {
                return BigDecimal.valueOf(4);
            } else if (score.compareTo(BigDecimal.valueOf(70)) >= 0) {
                return BigDecimal.valueOf(3);
            } else if (score.compareTo(BigDecimal.valueOf(60)) >= 0) {
                return BigDecimal.valueOf(2);
            } else if (score.compareTo(BigDecimal.valueOf(0)) > 0) {
                return BigDecimal.valueOf(1);
            } else {
                //英语分0分就是没考 可能是三年级以下
                return BigDecimal.valueOf(0);
            }
        } else {
            return BigDecimal.valueOf(0);
        }
    }


    /*查询一个班级*/
    @CrossOrigin
    @GetMapping("/class/avRateScore/{classId}/{timingId}")
    @ResponseBody
    public AjaxResult classAvRateScore(@PathVariable Long classId, @PathVariable Long timingId) {


        String type = "E";
        KxClasses kxClasses = kxClassesMapper.selectKxClassesById(classId);
        List<KxStudent> kxStudents = kxStudentMapper.selectKxStudentByClassId(classId);
//        KxTimingRate kxTimingRate = kxTimingRateMapper.selectKxTimingRateNow();
//        KxTiming kxTiming = kxTimingMapper.selectKxTimingByNow();
        KxTiming kxTiming = kxTimingMapper.selectKxTimingById(timingId);

        if (kxTiming == null) {
            return error("找不到该学期");
        }

        ArrayList<ScoreSort> scoreSorts = new ArrayList<>();
        for (KxStudent kxStudent : kxStudents) {
            ScoreSort scoreSort = kxScoreRecordMapper.selectKxScoreRecordBySort(kxStudent.getId(), type, 1L, kxTiming.getId());
            DecimalFormat df = new DecimalFormat("#.##"); // 创建 DecimalFormat 对象，指定格式为保留最多两位小数
            df.setRoundingMode(RoundingMode.DOWN); // 设置舍入模式为向下舍入
            float rounded;


            {
                float v = kxRateOftenMapper.selectKxScoreOftenAvRate(kxStudent.getId(), kxTiming.getId());
                rounded = Float.parseFloat(df.format(scoreSort.getScore() + v)); // 格式化小数部分并将结果转换为 float 类型
            }

            String result = df.format(rounded);
            scoreSort.setStudentName(kxStudent.getName());
            scoreSort.setImagePath(kxStudent.getImagePath());
            scoreSort.setScore(Float.parseFloat(result));
            KxMark kxMark = kxStudent.getKxMark();

            if (kxMark != null) {
                char firstChar = kxStudent.getClasses().getName().charAt(0);
                if (firstChar == '一' || firstChar == '二') {

                    scoreSort.setScore(changeFloat(scoreSort.getScore() + getRateMarkScore(kxMark.getMoralScore(), true).floatValue() + getRateMarkScore(kxMark.getChineseScore(), true).floatValue() + getRateMarkScore(kxMark.getMathScore(), true).floatValue() + getRateMarkScore(kxMark.getScienceScore(), true).floatValue() + getRateMarkScore(kxMark.getPhysicalScore(), true).floatValue() + getRateMarkScore(kxMark.getArtScore(), true).floatValue() + getRateMarkScore(kxMark.getMusicScore(), true).floatValue() + getRateMarkScore(kxMark.getLaoDongScore(), true).floatValue()
//                                       getRateMarkScore(kxMark.getEnglishScore(), true).floatValue()
                    ));


                } else {
                    logger.info("kxMark111:{}", kxMark.getMoralScore());

                    scoreSort.setScore(changeFloat(scoreSort.getScore() + getRateMarkScore(kxMark.getMoralScore(), false).floatValue() + getRateMarkScore(kxMark.getChineseScore(), false).floatValue() + getRateMarkScore(kxMark.getMathScore(), false).floatValue() + getRateMarkScore(kxMark.getScienceScore(), false).floatValue() + getRateMarkScore(kxMark.getPhysicalScore(), false).floatValue() + getRateMarkScore(kxMark.getArtScore(), false).floatValue() + getRateMarkScore(kxMark.getMusicScore(), false).floatValue() + getRateMarkScore(kxMark.getEnglishScore(), false).floatValue()));


                }

            }
            scoreSorts.add(scoreSort);
        }

        //只是排序一下
        scoreSorts.sort(new ScoreSort());

        float avScore = 0;
        for (ScoreSort scoreSort : scoreSorts) {
            avScore += scoreSort.getScore();
        }
        avScore = avScore / scoreSorts.size();
        return success().put("className", kxClasses.getName()).put("data", avScore).put("week", kxTiming);


    }

    /*查询一个年级的所有班的每个班的发展性平均分*/
    @CrossOrigin
    @GetMapping("/grade/avRateScore/{name}/{timingId}")
    @ResponseBody
    public AjaxResult gradeAvRateScore(@PathVariable String name, @PathVariable Long timingId) {
/*
        String type = "E";
        float avScore = 0;
        float size = 0;
//        KxTiming kxTiming = kxTimingMapper.selectKxTimingByNow();
        KxTiming kxTiming = kxTimingMapper.selectKxTimingById(timingId);

        if (kxTiming == null) {
            return error("找不到该学期");
        }

        List<KxClasses> kxClasses = kxClassesMapper.selectKxClassesByFirst(name);

        if (kxClasses == null && kxClasses.size() == 0) {
            return error("未找到对应班级");
        }

        for (KxClasses kxClass : kxClasses) {
            List<KxStudent> kxStudents = kxStudentMapper.selectKxStudentByClassId(kxClass.getId());
//        KxTimingRate kxTimingRate = kxTimingRateMapper.selectKxTimingRateNow();
            ArrayList<ScoreSort> scoreSorts = new ArrayList<>();
            for (KxStudent kxStudent : kxStudents) {
                ScoreSort scoreSort = kxScoreRecordMapper.selectKxScoreRecordBySort(kxStudent.getId(), type, 1L, kxTiming.getId());
                DecimalFormat df = new DecimalFormat("#.##"); // 创建 DecimalFormat 对象，指定格式为保留最多两位小数
                df.setRoundingMode(RoundingMode.DOWN); // 设置舍入模式为向下舍入
                float rounded;


                {
                    float v = kxRateOftenMapper.selectKxScoreOftenAvRate(kxStudent.getId(), kxTiming.getId());
                    rounded = Float.parseFloat(df.format(scoreSort.getScore() + v)); // 格式化小数部分并将结果转换为 float 类型
                }

                String result = df.format(rounded);
                scoreSort.setStudentName(kxStudent.getName());
                scoreSort.setImagePath(kxStudent.getImagePath());
                scoreSort.setScore(Float.parseFloat(result));
                KxMark kxMark = kxStudent.getKxMark();

                if (kxMark != null) {
                    char firstChar = kxStudent.getClasses().getName().charAt(0);
                    if (firstChar == '一' || firstChar == '二') {

                        scoreSort.setScore(changeFloat(scoreSort.getScore() + getRateMarkScore(kxMark.getMoralScore(), true).floatValue() + getRateMarkScore(kxMark.getChineseScore(), true).floatValue() + getRateMarkScore(kxMark.getMathScore(), true).floatValue() + getRateMarkScore(kxMark.getScienceScore(), true).floatValue() + getRateMarkScore(kxMark.getPhysicalScore(), true).floatValue() + getRateMarkScore(kxMark.getArtScore(), true).floatValue() + getRateMarkScore(kxMark.getMusicScore(), true).floatValue() + getRateMarkScore(kxMark.getLaoDongScore(), true).floatValue()
//                                       getRateMarkScore(kxMark.getEnglishScore(), true).floatValue()
                        ));


                    } else {
                        logger.info("kxMark111:{}", kxMark.getMoralScore());

                        scoreSort.setScore(changeFloat(scoreSort.getScore() + getRateMarkScore(kxMark.getMoralScore(), false).floatValue() + getRateMarkScore(kxMark.getChineseScore(), false).floatValue() + getRateMarkScore(kxMark.getMathScore(), false).floatValue() + getRateMarkScore(kxMark.getScienceScore(), false).floatValue() + getRateMarkScore(kxMark.getPhysicalScore(), false).floatValue() + getRateMarkScore(kxMark.getArtScore(), false).floatValue() + getRateMarkScore(kxMark.getMusicScore(), false).floatValue() + getRateMarkScore(kxMark.getEnglishScore(), false).floatValue()));


                    }

                }

                scoreSorts.add(scoreSort);
            }
            scoreSorts.sort(new ScoreSort());
            size+=scoreSorts.size();
            for (ScoreSort scoreSort : scoreSorts) {
                avScore += scoreSort.getScore();
            }
        }
        avScore = avScore / size;
        return success().put("data", avScore).put("week", kxTiming);
    */

        List<KxClasses> kxClasses = kxClassesMapper.selectKxClassesByFirst(name);
        ArrayList<AvRateClass> avScores = new ArrayList<>();
        KxTiming kxTiming = kxTimingMapper.selectKxTimingById(timingId);

        for (KxClasses kxClass : kxClasses) {
            String type = "E";
            List<KxStudent> kxStudents = kxStudentMapper.selectKxStudentByClassId(kxClass.getId());
//        KxTimingRate kxTimingRate = kxTimingRateMapper.selectKxTimingRateNow();
//        KxTiming kxTiming = kxTimingMapper.selectKxTimingByNow();

            if (kxTiming == null) {
                return error("找不到该学期");
            }

            ArrayList<ScoreSort> scoreSorts = new ArrayList<>();
            for (KxStudent kxStudent : kxStudents) {
                ScoreSort scoreSort = kxScoreRecordMapper.selectKxScoreRecordBySort(kxStudent.getId(), type, 1L, kxTiming.getId());
                DecimalFormat df = new DecimalFormat("#.##"); // 创建 DecimalFormat 对象，指定格式为保留最多两位小数
                df.setRoundingMode(RoundingMode.DOWN); // 设置舍入模式为向下舍入
                float rounded;


                {
                    float v = kxRateOftenMapper.selectKxScoreOftenAvRate(kxStudent.getId(), kxTiming.getId());
                    rounded = Float.parseFloat(df.format(scoreSort.getScore() + v)); // 格式化小数部分并将结果转换为 float 类型
                }

                String result = df.format(rounded);
                scoreSort.setStudentName(kxStudent.getName());
                scoreSort.setImagePath(kxStudent.getImagePath());
                scoreSort.setScore(Float.parseFloat(result));
                KxMark kxMark = kxStudent.getKxMark();

                if (kxMark != null) {
                    char firstChar = kxStudent.getClasses().getName().charAt(0);
                    if (firstChar == '一' || firstChar == '二') {

                        scoreSort.setScore(changeFloat(scoreSort.getScore() + getRateMarkScore(kxMark.getMoralScore(), true).floatValue() + getRateMarkScore(kxMark.getChineseScore(), true).floatValue() + getRateMarkScore(kxMark.getMathScore(), true).floatValue() + getRateMarkScore(kxMark.getScienceScore(), true).floatValue() + getRateMarkScore(kxMark.getPhysicalScore(), true).floatValue() + getRateMarkScore(kxMark.getArtScore(), true).floatValue() + getRateMarkScore(kxMark.getMusicScore(), true).floatValue() + getRateMarkScore(kxMark.getLaoDongScore(), true).floatValue()
//                                       getRateMarkScore(kxMark.getEnglishScore(), true).floatValue()
                        ));


                    } else {
                        logger.info("kxMark111:{}", kxMark.getMoralScore());

                        scoreSort.setScore(changeFloat(scoreSort.getScore() + getRateMarkScore(kxMark.getMoralScore(), false).floatValue() + getRateMarkScore(kxMark.getChineseScore(), false).floatValue() + getRateMarkScore(kxMark.getMathScore(), false).floatValue() + getRateMarkScore(kxMark.getScienceScore(), false).floatValue() + getRateMarkScore(kxMark.getPhysicalScore(), false).floatValue() + getRateMarkScore(kxMark.getArtScore(), false).floatValue() + getRateMarkScore(kxMark.getMusicScore(), false).floatValue() + getRateMarkScore(kxMark.getEnglishScore(), false).floatValue()));


                    }

                }
                scoreSorts.add(scoreSort);
            }
            scoreSorts.sort(new ScoreSort());

            float avScore = 0;
            for (ScoreSort scoreSort : scoreSorts) {
                avScore += scoreSort.getScore();
            }
            avScore = avScore / scoreSorts.size();
            avScores.add(new AvRateClass(avScore, kxClass.getName()));
        }
        return success().put("rows", avScores).put("week", kxTiming);
    }
}


# <body>
  <header>
    <div class="logo">My Github Page</div>
    <nav>
      <a href="#">Home</a>
      <a href="#">About</a>
      <a href="#">Contact</a>
    </nav>
  </header>
  <h1>Welcome to My Github Page</h1>
  <footer>&copy; 2023 My Github Page. All rights reserved.</footer>
</body>
</html>
