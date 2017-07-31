---
layout: post
title: "[C#] combobox显示数据库内容"
keywords: 
description: ""
category: 
tags: 
---

<!--markdown-->    SqlConnection Conn = //到sqlsever的连接          
                string Sql = "SELECT sno+'- '+sname as result FROM student where profession='" + comboBox1.SelectedItem + "'";
                DataSet Ds = new DataSet();  
                SqlDataAdapter Da = new SqlDataAdapter(Sql, Conn);  
                Da.Fill(Ds, "student");  
                comboBox2.DataSource = Ds.Tables["student"];  
                comboBox2.DisplayMember = "result";  
    
                Conn.Close();  
  
    在combobox1中添加以上触发事件  
  
    触发 combobox2 的显示内容为 sno - sname  
