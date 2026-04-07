import streamlit as st
import datetime
import pandas as pd

st.title("筋トレ記録")

#入力フォーム作る
with st.form("my_form"):
    year = st.number_input("年",value=2026)
    month = st.number_input("月",value=5,min_value=1,max_value=12)
    day = st.number_input("日",value=3,min_value=1,max_value=31)

    today = f"{year}/{month}/{day}"
    menu = st.text_input("種目は?","ベンチプレス")
    weight = st.number_input("重さは? (kg)", value=60)
    reps = st.number_input("回数は?", value=10)

    submitted = st.form_submit_button("記録する!")

if submitted:
    #データまとめる
    #today = datetime.date.today()
    new_data = pd.DataFrame([[today, menu,weight, reps]],
                            columns=['日付','種目','重さ','回数'])

    #csv に保存
    new_data.to_csv('kintore_log.csv', mode='a', header=False,index=False, encoding='utf-8')
    st.success(f"✅ {menu}の記録を保存しました")

st.subheader("これまでの記録")
try:
    df=pd.read_csv('kintore_log.csv',names=['日付','種目','重さ','回数'])
except:
    st.write("まだ記録がありません")


st.dataframe(df.iloc[::-1])

st.divider()
st.subheader("📈　成長の記録(グラフ)")
#種目を選べるようにする
target_menu = st.selectbox("グラフを表示する種目を選んでください",df['種目'].unique())

#選んだ種目だけのデータを取り出す
chart_data = df[df['種目'] == target_menu]

#日付順に並べて、日付を横軸にする
chart_data = chart_data.sort_values('日付')
st.line_chart(data=chart_data, x='日付',y='重さ')

st.divider()
st.subheader("データの削除")

if not df.empty:
    #選択しやすいように、日付と種目を組み合わせたリストを作
    df['display_name'] = df['日付'].astype(str) + " : " + df['種目'] + "(" + df['重さ'].astype(str) + "kg)"
    delete_target = st.selectbox("削除するデータを選んでください", df['display_name'].unique())

    if st.button("選択したデータを削除する"):
        #選択したデータ以外を残すことで削除を実現する
        df_new = df[df['display_name'] !=  delete_target]
        #余計な列を削除して保存
        df_new.drop(columns=['display_name']).to_csv('kintore_log.csv',index=False,header=False,mode='w',encoding='utf-8')
        st.warning("データを削除しました。反映するにはブラウザを更新してください")
        st.rerun()
