import pandas as pd
import streamlit as st
import matplotlib.pyplot as plt

# 페이지 설정
st.set_page_config(page_title="스포스탯 (SpoStat)", page_icon="🏃‍♂️", layout="wide")

# 한글 폰트 설정 (기본 폰트 사용)
plt.rcParams['font.family'] = 'NanumGothic' if plt.rcParams['font.family'] == ['sans-serif'] else plt.rcParams['font.family']
plt.rcParams['axes.unicode_minus'] = False

st.title("🏃‍♂️ AI 기반 스포츠센터 데이터 분석 서비스 (SpoStat)")
st.write("스포츠센터 이용 데이터를 업로드하여 주요 운영 지표와 시각화 분석 결과를 확인하세요.")

# 1. CSV 파일 업로드
uploaded_file = st.file_uploader("스포츠센터 이용 CSV 파일을 업로드하세요", type=["csv"])

if uploaded_file is not None:
    # 데이터 로드 및 전처리
    df = pd.read_csv(uploaded_file)
    df.columns = df.columns.str.strip()  # 컬럼명 공백 제거
    
    # 매출액 전처리 (쉼표 제거 후 정수 변환)
    if '매출액' in df.columns and df['매출액'].dtype == 'object':
        df['매출액'] = df['매출액'].astype(str).str.replace(',', '').str.strip().astype(int)
        
    st.success("데이터를 성공적으로 불러왔습니다!")

    # 2. 데이터 기본 정보 확인
    st.subheader("📌 데이터 기본 요약")
    col1, col2, col3, col4 = st.columns(4)
    col1.metric("전체 운영 건수", f"{len(df):,} 건")
    col2.metric("총 이용 인원", f"{df['이용인원'].sum():,} 명")
    col3.metric("총 매출액", f"{df['매출액'].sum():,} 원")
    col4.metric("평균 만족도", f"{df['만족도'].mean():.2f} / 5.0 점")

    st.markdown("---")

    # 3. 그래프 2개 이상
    st.subheader("📊 데이터 시각화")
    g_col1, g_col2 = st.columns(2)

    with g_col1:
        st.markdown("#### 1. 프로그램별 총 매출액")
        prog_sales = df.groupby('프로그램')['매출액'].sum().reset_index()
        fig1, ax1 = plt.subplots(figsize=(6, 4))
        ax1.bar(prog_sales['프로그램'], prog_sales['매출액'] / 10000, color='#4C72B0')
        ax1.set_ylabel("매출액 (만원)")
        ax1.set_title("프로그램별 매출 비교")
        plt.xticks(rotation=45)
        st.pyplot(fig1)

    with g_col2:
        st.markdown("#### 2. 시간대별 평균 만족도")
        time_sat = df.groupby('시간대')['만족도'].mean().reset_index()
        fig2, ax2 = plt.subplots(figsize=(6, 4))
        ax2.plot(time_sat['시간대'], time_sat['만족도'], marker='o', color='#DD8452', linewidth=2)
        ax2.set_ylabel("평균 만족도 (점)")
        ax2.set_title("시간대별 만족도 트렌드")
        st.pyplot(fig2)

    st.markdown("---")

    # 4. 주요 분석 결과
    st.subheader("💡 주요 데이터 분석 결과")
    st.info("""
    * **분석 결과 1 (수영 프로그램의 높은 매출 기여도):**
      전체 데이터 분석 결과, **수영** 프로그램이 가장 많은 이용 인원과 매출을 기록하며 스포츠센터의 핵심 수익원 역할을 하고 있습니다.
      
    * **분석 결과 2 (소규모 클래스의 높은 만족도):**
      **헬스PT** 및 **필라테스** 등 개인/소그룹 맞춤형 프로그램이 일반 강좌 대비 높은 만족도(평균 4.7점 이상)를 보이고 있어, 프리미엄 소그룹 클래스 확대를 추천합니다.
    """)

else:
    st.warning("상단에서 '중간고사_스포츠센터_이용데이터_1.csv' 파일을 업로드해주세요.")