# Dolphinsight Developer Guide, V1.0



# 1. About Dolphinsight

Dolphinsight(DI) is an interactive big data viz application which supports diverse data sources.
   - The target users for DI includes: data scientist, BI developer and anyone work with data and eager to find the data insight via simply interactive drag-n-drop manner.
   - Dolphinsight is developed by Dolphinsight Team(DIT). Please contact DIT via dolphinsight.bigdata@gmail.com .More information about Dolphinsight and DIT, please visit www.github.com/dolphinsight


# 2. Prerequisites

# 3. Dolphinsight Architecture

Currently, the DI is made up three key components and they are
   - dolphinsight(core), 
   - dolphinprotocol(protocol) and 
   - dolphinteractive(interactive).

The core holds all key modules for the DI as the backend.
The interactive is the UI frontend.
The protocol defines the "language" spoken between core and interactive, and it works quite simply and similar with Google Protocolbuffer.
The core is kind of micro-service.
# 4. Developer Must Know (DMK)

### 4.1 How to Add a New Viz Graph

As each component is highly modular and plugable, so ADDING a NEW VIZ GRAPH now becomes an easy task.Let's take adding simple scatter chart as example(named BasicScatterChart).
Following are the steps you could follow:

 - step 1. Add the VizGraphType, here is the BasicScatterChart.
```
package com.brill.dolphin.dolphinsight.graph.vizgraphtype;

public class VizGraphType {
    // all the graph types are listed here. with standard names
    public static final String ExampleChart = "ExampleChart";//only for example

    public static final String BasicLineChart = "BasicLineChart";// standard name

    public static final String StackAreaChart = "StackAreaChart"; // stack area chart

    public static final String BasicScatterChart = "BasicScatterChart"; // basic scatter chart
}
```
 - step 2. Add the graph data entity(for JSON representation) under graph/vizgraphtype/graph/echarts/stackareachart.
 
  At present, we work on ECharts to viz the data.The layout the entity should follow the ECharts object way.
e.g.
```
option = {
    xAxis: {
        scale: true
    },
    yAxis: {
        scale: true
    },
    series: [ {
        type: 'scatter',
        data: [[161.2, 51.6], [167.5, 59.0], [159.5, 49.2], [157.0, 63.0], [155.8, 53.6]      ],
    }]
};
```

The underlying design concept of this graph data entity is
a. Everything that related with visualization should encapsulated in a single 
JSON object and totally well prepared in the backend,
b. The frontend is an isolated/independent part that it just displays whatever
it given from the backend. The frontend just eats whatever it feed(the JSON object is the food).


Add the BasicScatterChart to represent the entity object.
```
package com.brill.dolphin.dolphinsight.graph.vizgraphtype;

import com.brill.dolphin.dolphinsight.graph.vizgraphtype.graph.echarts.basiclinechart.BasicLineChart;
import com.brill.dolphin.dolphinsight.graph.vizgraphtype.graph.echarts.basicscatterchart.BasicScatterChart;
import com.brill.dolphin.dolphinsight.graph.vizgraphtype.graph.echarts.stackareachart.StackAreaChart;

public class VizGraphBase {

    // VizGraphType
    private String vizGraphType;

    private BasicLineChart basicLineChart = null;
    private StackAreaChart stackAreaChart = null;
    private BasicScatterChart basicScatterChart = null;

    public VizGraphBase(String vizGraphType) {
        this.vizGraphType = vizGraphType;
    }

    public String getVizGraphType() {
        return vizGraphType;
    }

    public void setVizGraphType(String vizGraphType) {
        this.vizGraphType = vizGraphType;
    }

    public BasicLineChart getBasicLineChart() {
        return basicLineChart;
    }

    public void setBasicLineChart(BasicLineChart basicLineChart) {
        this.basicLineChart = basicLineChart;
    }

    public StackAreaChart getStackAreaChart() {
        return stackAreaChart;
    }

    public void setStackAreaChart(StackAreaChart stackAreaChart) {
        this.stackAreaChart = stackAreaChart;
    }

    public BasicScatterChart getBasicScatterChart() {
        return basicScatterChart;
    }

    public void setBasicScatterChart(BasicScatterChart basicScatterChart) {
        this.basicScatterChart = basicScatterChart;
    }
}

```
 - step 3. Add heuristic rule for this graph.
Whether the graph could be displayed or not, in an intelligent way, here we use heuristic rules to do that.
The rule inputs are: dimensions and measurements,
the output of the rule is whether the graph supports the combination of dimensions and measurements.
At first, we add the rule name
```
package com.brill.dolphin.dolphinsight.graph;

public class VizGraphRuleName {
    public static String Example = "Example";

    public static String BasicLineChart = "BasicLineChart"; // come from echarts
    public static String StackAreaChart = "StackAreaChart"; // come from echarts
    public static String BasicScatterChart = "BasicScatterChart"; // come for echarts
}
```
Then, create the rule for the graph, that is 
rule 1.only two dimensions and no measurements will trigger the BasicScatterChart, AND,
rule 2.the dimension should be all number
```
package com.brill.dolphin.dolphinsight.graph.vizgraphrule;

import com.brill.dolphin.dolphinsight.entity.model.DataSet;
import com.brill.dolphin.dolphinsight.entity.model.DataSource;
import com.brill.dolphin.dolphinsight.frontend.FunctionField;
import com.brill.dolphin.dolphinsight.graph.VizGraphRuleName;
import com.brill.dolphin.dolphinsight.graph.vizgraphtype.VizGraphType;
import com.brill.dolphin.dolphinsight.utils.TypeValidator;

import java.sql.Types;
import java.util.Vector;

public class SimpleScatterChartRule extends VizGraphRuleBase{

    public SimpleScatterChartRule() {
        super(VizGraphRuleName.BasicLineChart, VizGraphType.StackAreaChart);
    }
    public String getDescription() {
        return
                "        case 1:  d1, d2\n" +
                        "        select d1, d2 from t group by d1,d2 order by d1,d2 asc;\n" +
                        "        -axis: array of [d1,d2]\n" ;
    }

    public boolean isApplicable(DataSource dataSource, DataSet dataSet, Vector<FunctionField> dimensionFunctionFields, Vector<FunctionField> measurementFunctionFields) {
        boolean bApplicable = true;
        // the rule is:
        // for d1,d2
        // (1)|d|=2 and |m|=0
        // (2)d1,d2 should be number(int, float, double)

        if(dimensionFunctionFields.size() ==2 &&  measurementFunctionFields.size() ==0)
        {

            Vector<Integer> validDimensionDataTypes = new Vector<Integer>();

            validDimensionDataTypes.add(Types.INTEGER);
            validDimensionDataTypes.add(Types.NUMERIC);
            validDimensionDataTypes.add(Types.FLOAT);
            validDimensionDataTypes.add(Types.DECIMAL);
            validDimensionDataTypes.add(Types.DOUBLE);
            validDimensionDataTypes.add(Types.REAL);

            if(TypeValidator.isContainAnyDataType(dimensionFunctionFields, validDimensionDataTypes) == false)
            {
                bApplicable = false;
            }
        }
        else
        {
            bApplicable = false;
        }
        return bApplicable;

    
    }
}

```
After you created the rule as above, please add the rule into the VizGraphRuleEngine. The rule engine will scan each rule, and check if the rule is applicable.

```
package com.brill.dolphin.dolphinsight.graph;

import com.brill.dolphin.dolphinsight.entity.model.DataSet;
import com.brill.dolphin.dolphinsight.entity.model.DataSource;
import com.brill.dolphin.dolphinsight.frontend.FunctionField;
import com.brill.dolphin.dolphinsight.frontend.VizGraphSupported;
import com.brill.dolphin.dolphinsight.graph.vizgraphrule.*;

import java.util.Vector;

/**
 * use the rule engine to predict the graph should be supported.
 */
public class VizGraphRuleEngine {

    // hold all the viz graph type
    public static Vector<VizGraphRuleBase> Rules = new Vector<VizGraphRuleBase>();

    static
    {
        //Rules.add(new ExampleRule());
        // all new rules come to the below
        Rules.add(new BasicLineChartRule());
        Rules.add(new BasicScatterChartRule());
        Rules.add(new StackAreaChartRule());
    }

    /**
     * Give user the selection, return the viz graph supported.
     * @param dataSource
     * @param dataSet
     * @param columnFunctionFields
     * @param rowFunctionFields
     * @return
     */
    public VizGraphSupported getVizGraphSupported(DataSource dataSource,
                                                  DataSet dataSet,
                                                  Vector<FunctionField> columnFunctionFields,
                                                  Vector<FunctionField> rowFunctionFields) {
        VizGraphSupported vizGraphSupported = new VizGraphSupported();

        for (VizGraphRuleBase r : VizGraphRuleEngine.Rules) {
            if (r.isApplicable(dataSource, dataSet, columnFunctionFields, rowFunctionFields)) {
                vizGraphSupported.add(r.getVizGraphType());
            }
        }
        return vizGraphSupported;
    }
}

```

 - step 4. Add the planner for this graph.
The concept of the planner comes from the traditional database system.
The inputs of the planner are a.people's drag-n-drop from the frontend, and b.the graph type that user want to display .
More specific, the inputs includes (dimensions, mesurement, fitlers on dimensions and filters on measurements).
The outputs of the planner is the SQL to generate the entity mentioned above.

At first, we should add the PlanType:
```
package com.brill.dolphin.dolphinsight.executorengine.planner;

public class PlanType {

    public static String BasicLineChartPlan_d1m1="BasicLineChartPlan_d1m1";
    public static String BasicLineChartPlan_d1mn="BasicLineChartPlan_d1mn";

    public static String StackAreaChartPlan_d2m1="StackAreaChartPlan_d2m1";

    public static String BasicScatterChartPlan_d2m0 = "BasicScatterChartPlan_d2m0";
}
```
Then create the planner:
```
package com.brill.dolphin.dolphinsight.executorengine.planner.impl;

import com.brill.dolphin.dolphinsight.entity.model.DataSet;
import com.brill.dolphin.dolphinsight.entity.model.DataSource;
import com.brill.dolphin.dolphinsight.entity.model.UserPublicInfo;
import com.brill.dolphin.dolphinsight.executorengine.planner.Plan;
import com.brill.dolphin.dolphinsight.executorengine.planner.PlanType;
import com.brill.dolphin.dolphinsight.executorengine.planner.Planner;
import com.brill.dolphin.dolphinsight.executorengine.planner.StepType;
import com.brill.dolphin.dolphinsight.frontend.FieldFilter;
import com.brill.dolphin.dolphinsight.frontend.FunctionField;

import java.util.Vector;

public class BasicScatterChartPlanner extends Planner{
    public Plan generatePlan(UserPublicInfo user, DataSource dataSource, DataSet dataSet, String vizGraphType, Vector<FunctionField> dimensionFunctionFields, Vector<FunctionField> measurementFunctionFields, Vector<FieldFilter> fieldFilters) {
        Plan plan = new Plan(user, dataSource, dataSet, vizGraphType, dimensionFunctionFields, measurementFunctionFields, fieldFilters);

        /*
          "        case 1:  d1, d2\n" +
                        "        select d1, d2 from t where condition group by d1,d2 order by d1,d2 asc;\n" +
                        "        -axis: array of [d1,d2]\n" ;
         */

        plan.setPlanType(PlanType.BasicScatterChartPlan_d2m0);
        String lastSql = generateLastPlan(user, dataSource, dataSet, vizGraphType, dimensionFunctionFields, measurementFunctionFields, fieldFilters);

        plan.setSequenceStep(StepType.FinalStep, lastSql);

        return plan;
    }
}
```
Here, you can see the lastSql is the SQL to generate the entity object.

 - Step 5: Add the executor for the planner.
Please refer BasicScatterChartExecutor for the details.




### 4.2 How to Add a New Data Source
Dolphinsight is a viz application which aims to cover most of data sources,e.g. MySQL, PostgreSQL and others. Because of well architecture design , now Adding a New Data Source becomes quite easy. Following me.

 - Step 1: Add data source and data set
Most of work are in admin.service package. 

For each data source, there are two layers of concepts.One is DataSource, another is DataSet. In database domain, DataSource is the database, and the DataSet is the table(or relation). There are some predefined interfaces need to be implemented both in DataSource and DataSet.

For DataSource, there are two interfaces. 
a.verifyDataSource() is used to verify the connection/existence of data source(the existence of database).
b.listDataSetCandidateNames() is going to list all datasets(tables) in this datasource. Here, "candidate" means that we list all the tables within the database. 
```
package com.brill.dolphin.dolphinsight.admin.service;
public abstract class DataSourceOp {

    /**
     * Verify the connection of the data source
     * @param ip
     * @param port
     * @param databaseName
     * @param userName
     * @param password
     * @return
     */
    public abstract boolean verifyDataSource(String ip, Integer port, String databaseName, String userName, String password);

    /**
     * get the data set name list under this data source
     * @param dataSource
     * @return
     */
    public abstract Vector<String> listDataSetCandidateNames(DataSource dataSource);

}
```

For DataSet, there is only interface that need to be implemented.
regsiterDataSet() will register DataSet and the Field(a.k.a column) in DataSet into metadata store.

```
package com.brill.dolphin.dolphinsight.admin.service;
public abstract class DataSetOp {

    // init a dataSet entity according the data set name
    //public abstract DataSet initDataSet(DataSource dataSource, String dataSetName);
    public abstract DataSet registerDataSet(DataSource dataSource, String dataSetName);
}
```
Once you finished above to steps, please add newly created XXXDataSourceOp and XXXDataSetOp into OpManager.
```

package com.brill.dolphin.dolphinsight.admin.service;

import com.brill.dolphin.dolphinsight.admin.service.impl.mysql.MySQLDataSetOp;
import com.brill.dolphin.dolphinsight.admin.service.impl.mysql.MySQLDataSourceOp;
import com.brill.dolphin.dolphinsight.common.DataSourceType;

public class OpManager {
    public static DataSourceOp getDataSourceOp(String dataSourceType)
    {
        DataSourceOp dataSourceOp = null;

        if(dataSourceType.equalsIgnoreCase(DataSourceType.MySQL))
        {
            dataSourceOp = new MySQLDataSourceOp();
        }

        return dataSourceOp;
    }

    public static DataSetOp getDataSteOp(String dataSourceType)
    {
        DataSetOp dataSetOp = null;
        if(dataSourceType.equalsIgnoreCase(DataSourceType.MySQL))
        {
            dataSetOp = new MySQLDataSetOp();
        }

        return dataSetOp;
    }
}

```

 
 - Step 2:Add new data source entry
 ```
 package com.brill.dolphin.dolphinsight.common;

public class DataSourceType {

    public static String MySQL = "MySQL_Data_Source";
    public static String PostgreSQL = "PostgreSQL_Data_Source";
}

```
 - Step 3: Add new executor and query for data source
 
 The executor is the branch for generate the instance of executor of various graph executor.
 
 ```
 package com.brill.dolphin.dolphinsight.executorengine.executor.impl.postgresql;

import com.brill.dolphin.dolphinsight.executorengine.executor.Executor;
import com.brill.dolphin.dolphinsight.executorengine.executor.impl.BasicLineChartExecutor;
import com.brill.dolphin.dolphinsight.executorengine.executor.impl.BasicScatterChartExecutor;
import com.brill.dolphin.dolphinsight.executorengine.executor.impl.StackAreaChartExecutor;
import com.brill.dolphin.dolphinsight.executorengine.planner.Plan;
import com.brill.dolphin.dolphinsight.frontend.VizResult;
import com.brill.dolphin.dolphinsight.graph.vizgraphtype.VizGraphType;

public class PostgreSQLExecutor extends Executor{


    public VizResult executePlan(Plan plan) {

        VizResult vizResult = null;

        if(plan.getVizGraphType().equalsIgnoreCase(VizGraphType.BasicLineChart))
        {
            BasicLineChartExecutor basicLineChartExecutor = new BasicLineChartExecutor();
            vizResult = basicLineChartExecutor.executePlan(plan);
        }
        else if(plan.getVizGraphType().equalsIgnoreCase(VizGraphType.StackAreaChart))
        {
            StackAreaChartExecutor stackAreaChartExecutor = new StackAreaChartExecutor();
            vizResult = stackAreaChartExecutor.executePlan(plan);
        }
        else if(plan.getVizGraphType().equalsIgnoreCase(VizGraphType.BasicScatterChart))
        {
            BasicScatterChartExecutor basicScatterChartExecutor = new BasicScatterChartExecutor();
            vizResult = basicScatterChartExecutor.executePlan(plan);
        }


        return vizResult;
    }
}


 ```

The exmaple code of XXXQuery is as following.

```
package com.brill.dolphin.dolphinsight.executorengine.executor.impl.postgresql;

import com.brill.dolphin.dolphinsight.entity.model.DataSource;
import com.brill.dolphin.dolphinsight.executorengine.executor.QueryBase;
import com.brill.dolphin.dolphinsight.utils.ConnectionManager;

import java.sql.Connection;
import java.sql.ResultSet;
import java.sql.SQLException;
import java.sql.Statement;

public class PostgreSQLQuery implements QueryBase{

    private Connection connection = null;

    public boolean open(DataSource dataSource) {
        boolean bResult = false;

        connection =  ConnectionManager.getPostgreSQLConnection(dataSource.getIp(),
                Integer.valueOf(dataSource.getPort()),
                dataSource.getDatabaseName(),
                dataSource.getUserName(),
                dataSource.getPassword());

        bResult = (connection == null)? false : true;
        return bResult;
    }

    public ResultSet execute(String sqlStatement) {
        Statement statement = null;
        ResultSet resultSet = null;

        if(connection != null)
        {
            try {
                statement = connection.createStatement();
                System.out.println("Query Executed:");
                System.out.println(sqlStatement);
                resultSet = statement.executeQuery(sqlStatement);
            } catch (SQLException e) {
                if(statement != null) {
                    try {
                        statement.close();
                        statement = null;
                    } catch (SQLException e1) {
                        e1.printStackTrace();
                    }
                }

                if(resultSet != null)
                {
                    try {
                        resultSet.close();
                        resultSet = null;
                    } catch (SQLException e1) {
                        e1.printStackTrace();
                    }
                }
                e.printStackTrace();
            }
        }
        return resultSet;
    }

    public void close() {
        if(connection != null)
        {
            try {
                connection.close();
            } catch (SQLException e1) {
                e1.printStackTrace();
            }
        }
    }
}


```
 - Step 4: Regsiter the new executor and query for data source
 
 We need register the newly XXXExecutor and XXXQuery to ExecutorManager and QueryManager.
 
 ```
 package com.brill.dolphin.dolphinsight.executorengine.executor;

import com.brill.dolphin.dolphinsight.common.DataSourceType;
import com.brill.dolphin.dolphinsight.entity.model.DataSource;
import com.brill.dolphin.dolphinsight.executorengine.executor.impl.mysql.MySQLExecutor;
import com.brill.dolphin.dolphinsight.executorengine.executor.impl.postgresql.PostgreSQLExecutor;
import com.brill.dolphin.dolphinsight.executorengine.executor.impl.sqlserver.SQLServerExecutor;

public class ExecutorManager {


    public static Executor getExecutor(String dataSourceType)
    {
        Executor executor = null;

        if(dataSourceType.equalsIgnoreCase(DataSourceType.MySQL))
        {
            executor = new MySQLExecutor();
        }
        else if(dataSourceType.equalsIgnoreCase(DataSourceType.PostgreSQL))
        {
            executor = new PostgreSQLExecutor();
        }
        else if(dataSourceType.equalsIgnoreCase(DataSourceType.SQLServer))
        {
            executor = new SQLServerExecutor();
        }

        return executor;
    }
}
 ```
 and 
 
 ```
 package com.brill.dolphin.dolphinsight.executorengine.executor;

import com.brill.dolphin.dolphinsight.common.DataSourceType;
import com.brill.dolphin.dolphinsight.executorengine.executor.impl.mysql.MySQLQuery;
import com.brill.dolphin.dolphinsight.executorengine.executor.impl.postgresql.PostgreSQLQuery;
import com.brill.dolphin.dolphinsight.executorengine.executor.impl.sqlserver.SQLServerQuery;

public class QueryManager {

    public static QueryBase getQuery(String dataSourceType)
    {
        QueryBase query = null;

        if(dataSourceType.equalsIgnoreCase(DataSourceType.MySQL))
        {
            query = new MySQLQuery();
        }
        else if(dataSourceType.equalsIgnoreCase(DataSourceType.PostgreSQL))
        {
            query = new PostgreSQLQuery();
        }
        else if(dataSourceType.equalsIgnoreCase(DataSourceType.SQLServer))
        {
            query = new SQLServerQuery();
        }
        return query;
    }
}
 ```
