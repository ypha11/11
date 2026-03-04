# 11# 导入数据分析三剑客
import pandas as pd
import matplotlib.pyplot as plt
import numpy as np

# 设置中文字体（解决matplotlib图表中文乱码问题）
plt.rcParams['font.sans-serif'] = ['SimHei']
plt.rcParams['axes.unicode_minus'] = False

def student_score_analysis(csv_path):
    """
    学生成绩数据分析主函数
    :param csv_path: 学生成绩CSV文件路径（必填）
    """
    # 1. 读取数据（CSV格式，可替换为Excel格式：pd.read_excel()）
    print("="*50)
    print("1. 读取并查看数据基本信息")
    print("="*50)
    # 读取CSV文件，header=0表示第一行为表头（姓名、语文、数学、英语等）
    df = pd.read_csv(csv_path)
    # 查看数据前5行，快速了解数据结构
    print("数据前5行：")
    print(df.head())
    # 查看数据基本信息（行数、列数、数据类型、缺失值）
    print("\n数据基本信息：")
    print(df.info())
    # 查看数据统计描述（平均分、最高分、最低分、标准差等）
    print("\n数据统计描述：")
    print(df.describe())

    # 2. 数据清洗（处理缺失值、异常值，核心步骤）
    print("\n" + "="*50)
    print("2. 数据清洗（处理缺失值、异常值）")
    print("="*50)
    # 处理缺失值：将缺失值填充为对应科目的平均分（避免删除数据导致样本减少）
    # 筛选出成绩列（排除姓名等非数值列）
    score_cols = df.select_dtypes(include=[np.number]).columns
    for col in score_cols:
        # 计算该科目的平均分，round(2)保留2位小数
        mean_score = round(df[col].mean(), 2)
        # 填充缺失值
        df[col].fillna(mean_score, inplace=True)
        print(f"{col}科目缺失值已填充，填充值为：{mean_score}")
    
    # 处理异常值：成绩范围应在0-100之间，超出范围的视为异常值，替换为该科目的中位数
    for col in score_cols:
        # 找出异常值（<0 或 >100）
        abnormal_count = len(df[(df[col] < 0) | (df[col] > 100)])
        if abnormal_count > 0:
            # 计算中位数，用于替换异常值
            median_score = round(df[col].median(), 2)
            # 替换异常值
            df.loc[(df[col] < 0) | (df[col] > 100), col] = median_score
            print(f"{col}科目异常值（<0或>100）共{abnormal_count}个，已替换为中位数：{median_score}")
        else:
            print(f"{col}科目无异常值")

    # 3. 核心统计分析（提取有价值的信息）
    print("\n" + "="*50)
    print("3. 核心统计分析结果")
    print("="*50)
    # 计算各科目平均分、最高分、最低分、及格率（60分及格）
    analysis_result = []
    for col in score_cols:
        avg_score = round(df[col].mean(), 2)
        max_score = round(df[col].max(), 2)
        min_score = round(df[col].min(), 2)
        # 计算及格率：及格人数 / 总人数 * 100%
        pass_rate = round(len(df[df[col] >= 60]) / len(df) * 100, 2)
        analysis_result.append([col, avg_score, max_score, min_score, pass_rate])
    
    # 把分析结果转为DataFrame，便于查看
    result_df = pd.DataFrame(analysis_result, columns=["科目", "平均分", "最高分", "最低分", "及格率(%)"])
    print(result_df)

    # 计算总分，并添加到原数据中
    df["总分"] = df[score_cols].sum(axis=1)
    print(f"\n学生总分统计：")
    print(f"总分平均分：{round(df['总分'].mean(), 2)}")
    print(f"总分最高分：{round(df['总分'].max(), 2)}")
    print(f"总分最低分：{round(df['总分'].min(), 2)}")

    # 4. 数据可视化（生成直观图表，保存到本地）
    print("\n" + "="*50)
    print("4. 数据可视化（图表已保存到本地）")
    print("="*50)
    # 子图布局：2行2列，用于展示4个图表
    fig, axes = plt.subplots(2, 2, figsize=(12, 10))
    fig.suptitle("学生成绩数据分析图表", fontsize=16, fontweight="bold")

    # 图表1：各科目平均分柱状图
    axes[0, 0].bar(result_df["科目"], result_df["平均分"], color="#3498db", alpha=0.8)
    axes[0, 0].set_title("各科目平均分", fontsize=12)
    axes[0, 0].set_ylabel("平均分")
    # 在柱状图上添加数值标签
    for i, v in enumerate(result_df["平均分"]):
        axes[0, 0].text(i, v + 1, str(v), ha="center", va="bottom")

    # 图表2：各科目及格率柱状图
    axes[0, 1].bar(result_df["科目"], result_df["及格率(%)"], color="#2ecc71", alpha=0.8)
    axes[0, 1].set_title("各科目及格率", fontsize=12)
    axes[0, 1].set_ylabel("及格率(%)")
    for i, v in enumerate(result_df["及格率(%)"]):
        axes[0, 1].text(i, v + 1, str(v) + "%", ha="center", va="bottom")

    # 图表3：总分分布直方图（查看总分分布情况）
    axes[1, 0].hist(df["总分"], bins=10, color="#e74c3c", alpha=0.7, edgecolor="black")
    axes[1, 0].set_title("学生总分分布", fontsize=12)
    axes[1, 0].set_xlabel("总分")
    axes[1, 0].set_ylabel("学生人数")

    # 图表4：语文vs数学成绩散点图（查看两科目相关性）
    axes[1, 1].scatter(df["语文"], df["数学"], color="#9b59b6", alpha=0.6)
    axes[1, 1].set_title("语文vs数学成绩相关性", fontsize=12)
    axes[1, 1].set_xlabel("语文成绩")
    axes[1, 1].set_ylabel("数学成绩")
    # 添加趋势线（直观查看相关性）
    z = np.polyfit(df["语文"], df["数学"], 1)
    p = np.poly1d(z)
    axes[1, 1].plot(df["语文"], p(df["语文"]), "r--", alpha=0.8)

    # 调整子图间距，避免重叠
    plt.tight_layout()
    # 保存图表到本地（路径可修改，如"成绩分析图表.png"）
    plt.savefig("student_score_analysis.png", dpi=300, bbox_inches="tight")
    plt.close()
    print("图表已保存为：student_score_analysis.png")

# 主程序（运行代码时执行）
if __name__ == "__main__":
    # 替换为你的学生成绩CSV文件路径（示例路径，可修改）
    csv_file_path = "student_scores.csv"
    # 调用数据分析函数
    try:
        student_score_analysis(csv_file_path)
        print("\n" + "="*50)
        print("学生成绩数据分析完成！")
    except Exception as e:
        print(f"\n数据分析失败，错误原因：{str(e)}")
        print("提示：请检查CSV文件路径是否正确，文件格式是否为标准CSV（表头包含姓名、语文、数学等列）")
