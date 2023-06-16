为了实现该功能，我们可以使用 Java 编程语言编写代码来连接数据库，并定义一个 Score 实体类用于存储评分信息和提供相应的操作方法，如添加评分、查询所有评分等。具体的代码实现可以参考以下示例。

To achieve this functionality, we can write code in the Java programming language to connect to the database and define a Score entity class to store scoring information and provide corresponding action methods, such as adding scores, querying all scores, and so on. For specific code implementation, refer to the following example.


import java.sql.Connection;

import java.sql.DriverManager;

import java.sql.PreparedStatement;

import java.sql.ResultSet;

import java.sql.SQLException;


public class Score {

    private int id;
    
    private String studentName;
    
    private String className;
    
    private String evaluator;
    
    private String evaluation;
    
    private String date;
    

    // 构造函数
    public Score(int id, String studentName, String className, String evaluator, String evaluation, String date) {
        this.id = id;
        this.studentName = studentName;
        this.className = className;
        this.evaluator = evaluator;
        this.evaluation = evaluation;
        this.date = date;
    }

    // 添加评分
    public static void addScore(Score score) {
        Connection conn = null;
        PreparedStatement pstmt = null;

        try {
            // 连接数据库
            conn = DriverManager.getConnection("jdbc:mysql://localhost:3306/mydb", "root", "password");

            // 拼接 SQL 语句
            String sql = "INSERT INTO scores(student_name, class_name, evaluator, evaluation, date) VALUES (?, ?, ?, ?, ?)";
            pstmt = conn.prepareStatement(sql);

            // 设置参数
            pstmt.setString(1, score.getStudentName());
            pstmt.setString(2, score.getClassName());
            pstmt.setString(3, score.getEvaluator());
            pstmt.setString(4, score.getEvaluation());
            pstmt.setString(5, score.getDate());

            // 执行 SQL 语句
            pstmt.executeUpdate();

            System.out.println("添加评分成功！");
        } catch (SQLException e) {
            e.printStackTrace();
        } finally {
            // 关闭连接
            try {
                if (pstmt != null) pstmt.close();
                if (conn != null) conn.close();
            } catch (SQLException e) {
                e.printStackTrace();
            }
        }
    }

    // 查询所有评分
    public static Score[] getAllScores() {
        Connection conn = null;
        PreparedStatement pstmt = null;
        ResultSet rs = null;
        Score[] scores = null;

        try {
            // 连接数据库
            conn = DriverManager.getConnection("jdbc:mysql://localhost:3306/mydb", "root", "password");

            // 拼接 SQL 语句
            String sql = "SELECT * FROM scores";
            pstmt = conn.prepareStatement(sql);
            rs = pstmt.executeQuery();

            // 获取评分个数
            rs.last();
            int numRows = rs.getRow();
            scores = new Score[numRows];
            rs.beforeFirst();

            // 解析查询结果
            int i = 0;
            while (rs.next()) {
                int id = rs.getInt("id");
                String studentName = rs.getString("student_name");
                String className = rs.getString("class_name");
                String evaluator = rs.getString("evaluator");
                String evaluation = rs.getString("evaluation");
                String date = rs.getString("date");

                Score score = new Score(id, studentName, className, evaluator, evaluation, date);
                scores[i] = score;
                i++;
            }
        } catch (SQLException e) {
            e.printStackTrace();
        } finally {
            // 关闭连接
            try {
                if (rs != null) rs.close();
                if (pstmt != null) pstmt.close();
                if (conn != null) conn.close();
            } catch (SQLException e) {
                e.printStackTrace();
            }
        }

        return scores;
    }

    // getters and setters
    public int getId() {
        return id;
    }

    public void setId(int id) {
        this.id = id;
    }

    public String getStudentName() {
        return studentName;
    }

    public void setStudentName(String studentName) {
        this.studentName = studentName;
    }

    public String getClassName() {
        return className;
    }

    public void setClassName(String className) {
        this.className = className;
    }

    public String getEvaluator() {
        return evaluator;
    }

    public void setEvaluator(String evaluator) {
        this.evaluator = evaluator;
    }

    public String getEvaluation() {
        return evaluation;
    }

    public void setEvaluation(String evaluation) {
        this.evaluation = evaluation;
    }

    public String getDate() {
        return date;
    }

    public void setDate(String date) {
        this.date = date;
    }
}
