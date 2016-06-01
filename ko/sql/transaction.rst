데이터베이스 트랜잭션
=====================

데이터베이스 트랜잭션은 CUBRID 질의문을 일관성(다중 사용자 환경에서 유효한 결과를 만들어내는 것)과 복구(시스템 실패와 같은 어떤 장애에도 커밋된 트랜잭션의 결과를 유지하는 것과, 어떤 고장에도 불구하고 중단된 트랜잭션은 데이터베이스로부터 무효화되는 것을 보장하는 것)의 단위로 그룹화한다. 하나의 트랜잭션은 데이터베이스에 접근하고 갱신하는 하나의 질의문 또는 여러 질의문으로 구성된다.

CUBRID는 많은 사용자가 동시에 데이터베이스에 접근하도록 하고 데이터베이스의 불일치를 방지하기 위하여 사용자 간 접근과 갱신을 관리한다. 예를 들어 데이터가 한 사용자에 의해 갱신되었을 때 그 트랜잭션에 관련된 데이터의 변화는 갱신이 커밋될 때까지 다른 사용자나 데이터베이스에서 일어나는 다른 트랜잭션에 보이지 않는다. 트랜잭션이 커밋되지 않고 롤백될 수 있기 때문에 이 원칙은 중요하다.

트랜잭션 처리 결과를 확신할 때까지, 데이터베이스에 영구적으로 갱신하는 것을 연기할 수 있다. 또한 트랜잭션 처리 과정에서 응용 프로그램이나 컴퓨터 시스템에서 만족할 수 없는 결과나 실패가 발생하면 데이터베이스의 모든 갱신을 제거(**ROLLBACK**)할 수 있다. 트랜잭션의 끝은 **COMMIT WORK** 또는 **ROLLBACK WORK** 문으로 결정된다. **COMMIT WORK** 문은 데이터베이스의 모든 갱신을 영구적으로 만드는 반면에 **ROLLBACK WORK** 문은 트랜잭션에서 입력된 모든 갱신을 무효화시킨다.

트랜잭션 커밋
-------------

데이터베이스에서 일어난 갱신들은 **COMMIT WORK** 문이 주어지기 전까지 영구히 저장되지 않는다. "영구히(permanently)" 저장된다는 것은 디스크에 저장이 완료되는 것을 의미한다. 키워드 **WORK** 는 생략이 가능하다. 추가로 데이터베이스의 다른 사용자는 변경이 영구히 반영되기 전까지는 변경 사항을 볼 수 없다. 예를 들어 테이블에 새로운 행을 삽입했을 때 데이터베이스 트랜잭션이 커밋되기 전까지 그 행에 접근할 수 있는 것은 그 행을 삽입한 사용자뿐이다(**UNCOMMITTED INSTANCES** 격리 수준을 사용하면 다른 사용자가 일관성이 없는 커밋되지 않은 갱신을 볼 수도 있다).

트랜잭션이 커밋된 후에는 트랜잭션에서 획득한 모든 잠금이 해제된다. ::

    COMMIT [WORK];

.. code-block:: sql

    -- ;autocommit off
    -- AUTOCOMMIT IS OFF
    
    SELECT name, seats
    FROM stadium WHERE code IN (30138, 30139, 30140);

::

        name                                seats
    =============================================
        'Athens Olympic Tennis Centre'      3200
        'Goudi Olympic Hall'                5000
        'Vouliagmeni Olympic Centre'        3400

다음 **UPDATE** 문은 3개의 stadium의 seats 칼럼 값을 변경한다. 결과를 검토하기 위해 갱신이 일어나기 전에 현재의 값과 이름을 검색한다. 기본적으로 csql은 자동으로 autocommit으로 작동되므로, 예제에서는 autocommit 모드를 off로 설정한 후 동작을 시험한다.

.. code-block:: sql

    UPDATE stadium
    SET seats = seats + 1000
    WHERE code IN (30138, 30139, 30140);
     
    SELECT name, seats FROM stadium WHERE code in (30138, 30139, 30140);
    
::

        name                                seats
    ============================================
        'Athens Olympic Tennis Centre'      4200
        'Goudi Olympic Hall'                6000
        'Vouliagmeni Olympic Centre'        4400

만약 갱신이 제대로 이루어 졌다면 변경을 영구적으로 만들 수 있다. 이때 아래처럼 **COMMIT WORK** 문을 사용한다.

.. code-block:: sql

    COMMIT [WORK];

.. note:: CUBRID에서는 트랜잭션 처리 시 기본적으로 자동 커밋 모드로 지정된다.

자동 커밋 모드는 모든 SQL 문을 자동으로 커밋 또는 롤백하는 모드로서, 해당 SQL 문이 정상 수행되면 해당 트랜잭션을 자동 커밋하고, 오류가 발생하면 트랜잭션을 롤백한다.

이러한 자동 커밋 모드는 모든 인터페이스에서 지원되며, CCI, PHP, ODBC, OLE DB 인터페이스는 브로커 파라미터인 **CCI_DEFAULT_AUTOCOMMIT** 을 통해 응용 프로그램 시작 시의 자동 커밋 모드를 설정할 수 있다. 브로커 파라미터 설정이 생략될 경우 기본값은 **ON** 이다. CCI 인터페이스는 **cci_set_autocommit** (), PHP 인터페이스는 **cubrid_set_autocommit** () 함수를 이용하여 응용 프로그램 내에서 자동 커밋 모드 설정 여부를 변경할 수 있다. 

CSQL 인터프리터에서 자동 커밋 모드를 설정하는 세션 명령어(**;AUtocommit**)에 대해서는 :ref:`csql-session-commands` 를 참조한다.

트랜잭션 롤백
-------------

**ROLLBACK WORK** 문은 마지막 트랜잭션 이후의 모든 데이터베이스의 갱신을 제거한다. **WORK** 키워드는 생략 가능하다. 이것은 데이터베이스에 영구적으로 입력하기 전에 부정확하고 불필요한 갱신을 무효화할 수 있다. 트랜잭션 동안 획득한 모든 잠금은 해제된다. ::

    ROLLBACK [WORK];

다음 예제는 동일한 테이블의 정의와 행을 수정하는 두 개의 명령을 보여주고 있다.

.. code-block:: sql

    -- csql> ;autocommit off
    CREATE TABLE code2 (
        s_name  CHAR(1),
        f_name  VARCHAR(10)
    );
    COMMIT;
    
    ALTER TABLE code2 DROP s_name;
    INSERT INTO code2 (s_name, f_name) VALUES ('D','Diamond');
 
::

    ERROR: s_name is not defined.

*code* 테이블의 정의에서 *s_name* 칼럼이 이미 제거되었기 때문에 **INSERT** 문의 실행은 실패한다. *code* 테이블에 입력하려고 했던 데이터는 틀리지 않으나 테이블에서 칼럼이 잘못 제거되었다. 이 시점에서 *code* 테이블의 원래 정의를 복원하기 위해서 **ROLLBACK WORK** 문을 사용할 수 있다.

.. code-block:: sql

    ROLLBACK WORK;

이후에 **ALTER CLASS** 명령을 다시 입력하여 *s_name* 칼럼을 제거하며, **INSERT** 문을 수정한다. 트랜잭션이 중단되었기 때문에 **INSERT** 명령은 다시 입력되어야 한다. 데이터베이스 갱신이 의도한 대로 이루어졌으면 변경을 영구화하기 위해 트랜잭션을 커밋한다.

.. code-block:: sql

    ALTER TABLE code DROP s_name;
    INSERT INTO code (f_name) VALUES ('Diamond');

    COMMIT WORK;

세이브포인트와 부분 롤백
------------------------

세이브포인트(savepoint)는 트랜잭션이 진행되는 중에 수립되는데, 트랜잭션에 의해 수행되는 데이터베이스 갱신을 세이브포인트 지점까지만 롤백할 수 있도록 하기 위해서이다. 이 연산을 부분 롤백(partial rollback)이라고 부른다. 부분 롤백에서는 세이브포인트 이후의 데이터베이스 연산(삽입, 삭제, 갱신 등)은 하지 않은 것으로 되고 세이브포인트 지점을 포함하여 앞서 진행된 트랜잭션의 연산은 그대로 유지된다. 부분 롤백이 실행된 후에 트랜잭션은 다른 연산을 계속 진행할 수 있다. 또는 **COMMIT WORK** 문이나 **ROLLBACK WORK** 문으로 트랜잭션을 끝낼 수도 있다. 세이브포인트는 트랜잭션에서 수행된 갱신을 커밋하는 것이 아님을 명심해야 한다.

세이브포인트는 트랜잭션의 어느 시점에서도 만들 수 있고 몇 개의 세이브포인트라도 어떤 주어진 시점에 사용될 수 있다. 특정 세이브포인트보다 앞선 세이브포인트로 부분 롤백이 수행되거나 **COMMIT WORK** 또는 **ROLLBACK WORK** 문으로 트랜잭션이 끝나면 특정 세이브포인트는 제거된다. 특정 세이브포인트 이후에 대한 부분 롤백은 여러 번 수행될 수 있다.

세이브포인트는 길고 복잡한 프로그램을 통제할 수 있도록 중간 단계를 만들고 이름을 붙일 수 있기 때문에 유용하다. 예를 들어, 많은 갱신 연산 수행 시 세이브포인트를 사용하면 실수를 했을 때 모든 문장을 다시 수행할 필요가 없다. ::

    SAVEPOINT mark;
    mark:
    _ a SQL identifier
    _ a host variable (starting with :)

같은 트랜잭션 내에 여러 개의 세이브포인트를 지정할 때 *mark* 를 같은 값으로 하면 마지막 세이브포인트만 부분 롤백에 나타난다. 그리고 앞의 세이브포인트는 제일 마지막 세이브포인트로 부분 롤백할 때까지 감춰졌다가 제일 마지막 세이브포인트가 사용된 후 없어지면 나타난다. ::

    ROLLBACK [WORK] [TO [SAVEPOINT] mark] ;
    mark:
    _ a SQL identifier
    _ a host variable (starting with :)

앞에서는 **ROLLBACK WORK** 문이 마지막 트랜잭션 이후로 입력된 모든 데이터베이스의 갱신을 제거하였다. **ROLLBACK WORK** 문은 특정 세이브포인트 이후로 트랜잭션의 갱신을 되돌리는 부분 롤백에도 사용된다.

*mark* 의 값이 주어지지 않으면 트랜잭션은 모든 갱신을 취소하면서 종료한다. 여기에는 트랜잭션에 만들어진 모든 세이브포인트도 포함한다. *mark* 가 주어지면 지정한 세이브포인트 이후의 것은 취소되고, 세이브포인트를 포함한 이전의 것은 갱신 사항이 남는다.

다음 예제는 트랜잭션의 일부를 롤백하는 방법을 보여준다.
먼저 savepoint SP1, SP2를 설정한다.

.. code-block:: sql

    -- csql> ;autocommit off
    
    CREATE TABLE athlete2 (name VARCHAR(40), gender CHAR(1), nation_code CHAR(3), event VARCHAR(30));
    INSERT INTO athlete2(name, gender, nation_code, event)
    VALUES ('Lim Kye-Sook', 'W', 'KOR', 'Hockey');
    SAVEPOINT SP1;
     
    SELECT * from athlete2;
    INSERT INTO athlete2(name, gender, nation_code, event)
    VALUES ('Lim Jin-Suk', 'M', 'KOR', 'Handball');
     
    SELECT * FROM athlete2;
    SAVEPOINT SP2;
     
    RENAME TABLE athlete2 AS sportsman;
    SELECT * FROM sportsman;
    ROLLBACK WORK TO SP2;

위에서 *athlete2* 테이블의 이름 변경은 위의 부분 롤백에 의해서 롤백된다. 다음의 문장은 원래의 이름으로 질의를 수행하여 이것을 검증하고 있다.

.. code-block:: sql

    SELECT * FROM athlete2;
    DELETE FROM athlete2 WHERE name = 'Lim Jin-Suk';
    SELECT * FROM athlete2;
    ROLLBACK WORK TO SP2;

위에서 'Lim Jin-Suk' 을 삭제한 것은 이후에 진행되는 rollback work to SP2 명령문에 의해서 취소되었다.
다음은 SP1으로 롤백하는 경우이다.

.. code-block:: sql

    SELECT * FROM athlete2;
    ROLLBACK WORK TO SP1;
    SELECT * FROM athlete2;
    COMMIT WORK;

.. _cursor-holding:

커서 유지
=========

응용 프로그램이 명시적인 커밋 혹은 자동 커밋 이후에도 **SELECT** 질의 결과의 레코드셋을 유지하여 다음 레코드를 읽을(fetch) 수 있도록 하는 것을 커서 유지(cursor holdability)라고 한다. 각 응용 프로그램에서 연결 수준(connection level) 또는 문장 수준(statement level)으로 커서 유지 기능을 설정할 수 있으며, 설정을 명시하지 않으면 기본으로 커서가 유지된다.

다음 코드는 JDBC에서 커서 유지를 설정하는 예이다.

.. code-block:: java

    // set cursor holdability at the connection level
    conn.setHoldability(ResultSet.HOLD_CURSORS_OVER_COMMIT);
     
    // set cursor holdability at the statement level which can override the connection
    PreparedStatement pStmt = conn.prepareStatement(sql,
                                        ResultSet.TYPE_SCROLL_SENSITIVE,
                                        ResultSet.CONCUR_UPDATABLE,
                                        ResultSet.HOLD_CURSORS_OVER_COMMIT);

커밋 시점에 커서를 유지하지 않고 커서를 닫도록 설정하고 싶으면, 위의 예제에서 **ResultSet.HOLD_CURSORS_OVER_COMMIT** 대신 **ResultSet.CLOSE_CURSORS_AT_COMMIT** 를 설정한다.

CCI 로 개발된 응용 프로그램 역시 커서 유지가 기본 동작이며, 연결 수준에서 커서를 유지하지 않도록 설정한 경우 질의를 prepare할 때 **CCI_PREPARE_HOLDABLE** 플래그를 명시하면 해당 질의 수준에서 커서를 유지한다. CCI로 개발된 드라이버(PHP, PDO, ODBC, OLE DB, ADO.NET, Perl, Python, Ruby) 역시 커서 유지가 기본 동작이며, 커서 유지 여부의 설정을 지원하는지에 대해서는 해당 드라이버의 **PREPARE** 함수를 참고한다.

.. note:: \

    *   CUBRID 9.0 미만 버전까지는 커서 유지를 지원하지 않으며, 커밋이 발생하면 커서가 자동으로 닫히는 것이 기본 동작이다.
    *   CUBRID는 현재 java.sql.XAConnection 인터페이스에서 ResultSet.HOLD_CURSORS_OVER_COMMIT을 지원하지 않는다.

**트랜잭션 종료 시의 커서 관련 동작**

트랜잭션이 커밋되면 커서 유지로 설정되어 있더라도 모든 잠금은 해제된다.

트랜잭션이 롤백되면 결과 셋이 닫힌다. 이것은 커서 유지가 설정되어 현재 트랜잭션에서 유지되던 결과 셋이 닫힌다는 것을 의미한다.

.. code-block:: java

    rs1 = stmt.executeQuery(sql1);
    conn.commit();
    rs2 = stmt.executeQuery(sql2);
    conn.rollback();  // 결과 셋 rs2와 rs1이 닫히게 되어 둘 다 사용하지 못하게 됨.

**결과 셋이 종료되는 경우**

커서가 유지되는 결과 셋은 다음의 경우에 닫힌다.

*   드라이버에서 결과 셋을 닫는 경우(예: rs.close() 등)
*   드라이버에서 statement를 닫는 경우(예: stmt.close() 등)
*   드라이버 연결 종료
*   트랜잭션을 롤백하는 경우(예: 자동 커밋 OFF 모드에서 사용자의 명시적인 롤백 호출, 자동 커밋 ON 모드에서 질의 실행 오류 발생 등)

**CAS와의 관계**

응용 프로그램에서 커서 유지로 설정되어 있다고 해도 응용 프로그램과 CAS와의 연결이 끊기면 결과 셋은 자동으로 닫힌다. 브로커 파라미터인 **KEEP_CONNECTION** 의 설정 값은 결과 셋의 커서 유지에 영향을 미친다.

*   KEEP_CONNECTION = ON: 커서 유지에 영향을 주지 않음.
*   KEEP_CONNECTION = AUTO: 커서 유지되는 결과 셋이 열려 있는 동안 CAS가 재시작될 수 없음.

.. warning:: 결과 셋을 닫지 않은 상태로 유지하는 만큼 메모리 사용량이 늘어날 수 있으므로 사용을 마친 결과 셋은 반드시 닫아야 한다.

.. note:: CUBRID 9.0 미만 버전까지는 커서 유지를 지원하지 않으며, 커밋이 발생하면 커서가 자동으로 닫힌다. 즉, **SELECT** 질의 결과의 레코드셋을 유지하지 않는다.


.. _database-concurrency:

데이터베이스 동시성
===================

다수의 사용자들이 데이터베이스에서 읽고 쓰는 권한을 가질 때, 한 명 이상의 사용자가 동시에 같은 데이터에 접근할 가능성이 있다. 데이터베이스의 무결성을 보호하고, 사용자와 트랜잭션이 항상 정확하고 일관된 데이터를 지니기 위해서는 다중 사용자 환경에서의 접근과 갱신에 대한 통제가 필수적이다. 적정한 통제가 없으면 데이터는 어긋난 순서로 부정확하게 갱신될 수 있다.

대부분의 상용 데이터베이스 시스템과 마찬가지로 CUBRID도 데이터베이스 내의 동시성(concurrency)을 위한 기본 요소인 직렬성(serializability)을 수용한다. 직렬성이란 여러 트랜잭션이 동시에 수행될 때, 마치 각각의 트랜잭션이 순차적으로 수행되는 것처럼 트랜잭션 간 간섭이 없다는 것을 의미하며, 트랜잭션의 격리 수준(isolation level)이 높을수록 보장된다. 이러한 원칙은 원자성(atomic, 트랜잭션의 모든 영향들은 커밋되거나 롤백되어야 함)을 갖는 트랜잭션이 각각 수행된다면, 데이터베이스의 동시성이 보장된다는 가정에 기초하고 있다. CUBRID에서 직렬성은 잘 알려진 2단계 잠금(two-phase locking)  기법을 통해 관리된다.

커밋하고자 하는 트랜잭션은 데이터베이스의 동시성을 보장하고, 적합한 결과를 보장해야 한다. 여러 트랜잭션이 동시에 수행 중일 때, 트랜잭션 T1 내의 이벤트는 트랜잭션 T2에 영향을 끼치지 않아야 하며, 이를 격리성(isolation)이라 한다. 즉, 트랜잭션의 격리 수준(isolation level)은 동시에 수행되는 다른 트랜잭션으로부터 간섭받는 것을 허용하는 정도의 단위이다. 격리 수준이 높을수록 트랜잭션 간 간섭이 적으며 직렬적이고, 격리 수준이 낮을수록 트랜잭션 간 간섭이 많고 병렬적이며 동시성이 높아진다. 이러한 트랜잭션의 격리 수준에 따라 데이터베이스는 테이블과 레코드에 대해 어떤 잠금을 획득할지 결정한다. 따라서, 적용하고자 하는 서비스의 특성에 따라 격리 수준을 적절히 설정함으로써 데이터베이스의 일관성(consistency)과 동시성(concurrency)을 조정할 수 있다.

트랜잭션 격리 수준 설정을 통해 트랜잭션 간 간섭을 허용할 수 있는 읽기 연산의 종류는 다음과 같다.

*   **더티 읽기** (dirty read): 트랜잭션 T1이 데이터 D를 D'으로 갱신한 후 커밋을 수행하기 전에 트랜잭션 T2가 D'을 읽을 수 있다.
*   **반복할 수 없는 읽기** (non-repeatable read, unrepeatable read): 트랜잭션 T1이 데이터를 반복 조회하는 중에 다른 트랜잭션 T2가 데이터를 갱신 혹은 삭제하고 커밋하는 경우, 트랜잭션 T1은 수정된 값을 읽을 수 있다.
*   **유령 읽기** (phantom read): 트랜잭션 T1에서 데이터를 여러 번 조회하는 중에 다른 트랜잭션 T2가 새로운 레코드 E를 삽입하고 커밋한 경우, 트랜잭션 T1은 E를 읽을 수 있다.

CUBRID에서 트랜잭션 격리 수준의 기본 설정은 :ref:`isolation-level-4`\이다.

**CUBRID가 제공하는 격리 수준**

아래 표에서 격리 수준 옆에 있는 괄호 안의 숫자는 격리 수준을 설정할 때 격리 수준 명칭 대신 사용할 수 있는 번호이다.

사용자는 :ref:`set-transaction-isolation-level` 문을 사용하거나 CUBRID가 지원하는 동시성/잠금 파라미터를 이용하여 격리 수준을 설정할 수 있는데, 이에 관한 설명은 :ref:`lock-parameters`\ 를 참조한다.

(O: YES, X: NO)

+--------------------------------+--------+-----------+--------+----------------------+
| CUBRID 격리 수준               | 더티   | 반복할 수 | 유령   | 조회 중인 테이블에   |
| (isolation_level)              | 읽기   | 없는 읽기 | 읽기   | 대한 스키마 갱신     |
+================================+========+===========+========+======================+
| :ref:`isolation-level-6` (6)   | X      | X         | X      | X                    |
+--------------------------------+--------+-----------+--------+----------------------+
| :ref:`isolation-level-5` (5)   | X      | X         | O      | X                    |
+--------------------------------+--------+-----------+--------+----------------------+
| :ref:`isolation-level-4` (4)   | X      | O         | O      | X                    |
+--------------------------------+--------+-----------+--------+----------------------+

*   **READ COMMITTED**\는 더티 읽기(dirty read)를 불허하며 반복할 수 없는 읽기(unrepeatable read), 유령 읽기(phantom read)를 허용한다.
*   **REPEATABLE READ**\는 더티 읽기(dirty read), 반복할 수 없는 읽기(unrepeatable read)를 불허하며 유령 읽기(phantom read)를 허용한다.
*   **SERIALIZABLE**\은 읽기 연산 시 트랜잭션 간 간섭을 불허한다.


## gichoi ##
.. _mvcc-snapshot:

Multiversion Concurrency Control
================================

CUBRID previous versions managed isolation levels using the well known two phase locking protocol. In this protocol, a transaction obtains a shared lock before it reads an object, and an exclusive lock before it updates the object so that conflicting operations are not executed simultaneously. If transaction *T1* requires a lock, the system checks if the requested lock conflicts with the existing one. If it does, transaction *T1* enters a standby state and delays the lock. If another transaction *T2* releases the lock, transaction *T1* resumes and obtains it. Once the lock is released, the transaction do not require any new locks. 

CUBRID 10.0 replaced the two phase locking protocol with a Multiversion Concurrency Control (MVCC) protocol. Unlike two phase locking, MVCC does not block readers to access objects being modified by concurrent transactions. MVCC duplicates rows, creating multiple versions on each update. Never blocking readers is important for workloads involving mostly value reads from the database, commonly used in read-world scenarios. Exclusive locks are still required before updating objects. 

MVCC is also known for providing point in time consistent view of the database and for being particularly adept at implementing true **snapshot isolation** with less performance costs than other methods of concurrency.

Versioning, visibility and snapshot
-----------------------------------

MVCC maintains multiple versions for each database row. Each version is marked by its inserter and deleter with MVCCID's - unique identifiers for writer transactions. These markers are useful to identify the author of a change and to place the change on a timeline.

When a transaction *T1* inserts a new row, it creates its first version and sets its unique identifier *MVCCID1* as insert id. The MVCCID is stored as meta-data in record header:

.. image:: /images/transaction_inserted_record.png

Until *T1* commits, other transactions should not see this row. The MVCCID helps identifying the authors of database changes and place them on a time line, so others can know if the change is valid or not. In this case, anyone checking this row find the *MVCCID1*, find out that the owner is still active, hence the row must be (still) invisible.

After *T1* commits, a new transaction *T2* finds the row and decides to remove it. *T2* does not remove this version, allowing others to access it, instead it gets an exclusive lock, to prevent others from changing it, and marks the version as deleted. It adds another MVCCID so others can identify the deleter:

.. image:: /images/transaction_deleted_record.png

If *T2* decides instead to update one of the record values, it must create a new version. Both versions are marked with transaction MVCCID, old version for delete and new version for insert. Old version also stores a link to the location of new version, and the row representations looks like this:

.. image:: /images/transaction_updated_record.png

Currently, only *T2* can see second row version, while all other transaction will continue to access the first version. The property of a version to be seen or not to be seen by running transactions is called **visibility**. The visibility property is relative to each transaction, some can consider it true, whereas others can consider it false.

A transaction *T3* that starts after *T2* executes row update, but before *T2* commits, will not be able to see its new version, not even after *T2* commits. The visibility of one version towards *T3* depends on the state of its inserter and deleter when *T3* started and preserves its status for the lifetime of *T3*.

As a matter of fact, the visibility of all versions in database towards on transaction does not depend on the changes that occur after transaction is started. Moreover, any new version added is also ignored. Consequently, the set of all visible versions in the database remains unchanged and form the snapshot of the transaction. Hence, **snapshot isolation** is provided by MVCC and it is a guarantee that all read queries made in a transaction see a consistent view of the database.

In CUBRID 10.0, **snapshot** is a filter of all invalid MVCCID's. An MVCCID is invalid if it is not committed before the snapshot is taken. To avoid updating the snapshot filter whenever a new transaction starts, the snapshot is defined using two border MVCCID's: the lowest active MVCCID and the highest committed MVCCID. Only a list of active MVCCID values between the border is saved. Any transaction starting after snapshot is guaranteed to have an MVCCID bigger than highest committed and is automatically considered invalid. Any MVCCID below lowest active must be committed and is automatically considered valid.

The snapshot filter algorithm that decides a version visibility queries the MVCCID markers used for insert and delete:

+--------------------------------------+-----------------------------------------------------+
|                                      | Delete MVCCID                                       |
|                                      +--------------------------+--------------------------+
|                                      | Valid                    | Invalid                  |
+--------------------+-----------------+--------------------------+--------------------------+
| Insert MVCCID      | Valid           | Not visible              | Visible                  |
|                    +-----------------+--------------------------+--------------------------+
|                    | Invalid         | Impossible               | Not visible              |
+--------------------+-----------------+--------------------------+--------------------------+

Table explained:

*   **Valid insert MVCCID, valid delete MVCCID:** Version was inserted and deleted (and both committed) before snapshot, hence it is not visible.
*   **Valid insert MVCCID, invalid delete MVCCID:** Version was inserted and committed, but it was not deleted or it was recently deleted, hence it is visible.
*   **Invalid insert MVCCID, invalid delete MVCCID:** Inserter did not commit before snapshot, and record is not deleted or delete was also not committed, hence version is not visible.
*   **Invalid insert MVCCID, valid delete MVCCID:** Inserter did not commit and deleter did - impossible case. If deleter is not the same as inserter, it could not see the version, and if it is, both insert and delete MVCCID must be valid or invalid.

Let's see how snapshot works (**REPEATABLE READ** isolation will be used to keep same snapshot during entire transaction):
**Example 1: Inserting a new row**

+-------------------------------------------------------------------+-----------------------------------------------------------------------------------+
| session 1                                                         | session 2                                                                         |
+===================================================================+===================================================================================+
| .. code-block:: sql                                               | .. code-block:: sql                                                               |
|                                                                   |                                                                                   |
|   csql> ;autocommit off                                           |   csql> ;autocommit off                                                           |
|                                                                   |                                                                                   |
|   AUTOCOMMIT IS OFF                                               |   AUTOCOMMIT IS OFF                                                               |
|                                                                   |                                                                                   |
|   csql> set transaction isolation level REPEATABLE READ;          |   csql> set transaction isolation level REPEATABLE READ;                          |
|                                                                   |                                                                                   |
|   Isolation level set to:                                         |   Isolation level set to:                                                         |
|   REPEATABLE READ                                                 |   REPEATABLE READ                                                                 |
|                                                                   |                                                                                   |
+-------------------------------------------------------------------+-----------------------------------------------------------------------------------+
--------------------------------+
| .. code-block:: sql                                               |                                                                                   |
|                                                                   |                                                                                   |
|   csql> CREATE TABLE tbl(host_year integer, nation_code char(3)); |                                                                                   |
|   csql> COMMIT WORK;                                              |                                                                                   |
|                                                                   |                                                                                   |
+-------------------------------------------------------------------+-----------------------------------------------------------------------------------+
| .. code-block:: sql                                               |                                                                                   |
|                                                                   |                                                                                   |
|   -- insert a row without committing                              |                                                                                   |
|   csql> INSERT INTO tbl VALUES (2008, 'AUS');                     |                                                                                   |
|                                                                   |                                                                                   |
|   -- current transaction sees its own changes                     |                                                                                   |
|   csql> SELECT * FROM tbl;                                        |                                                                                   |
|                                                                   |                                                                                   |
|       host_year  nation_code                                      |                                                                                   |
|   ===================================                             |                                                                                   |
|            2008  'AUS'                                            |                                                                                   |
+-------------------------------------------------------------------+-----------------------------------------------------------------------------------+
|                                                                   | .. code-block:: sql                                                               |
|                                                                   |                                                                                   |
|                                                                   |   -- this snapshot should not see uncommitted row                                 |
|                                                                   |   csql> SELECT * FROM tbl;                                                        |
|                                                                   |                                                                                   |
|                                                                   |   There are no results.                                                           |
|                                                                   |                                                                                   |
+-------------------------------------------------------------------+-----------------------------------------------------------------------------------+
| .. code-block:: sql                                               |                                                                                   |
|                                                                   |                                                                                   |
|   csql> COMMIT WORK;                                              |                                                                                   |
|                                                                   |                                                                                   |
+-------------------------------------------------------------------+-----------------------------------------------------------------------------------+
|                                                                   | .. code-block:: sql                                                               |
|                                                                   |                                                                                   |
|                                                                   |   -- even though inserter did commit, this snapshot still can't see the row       |
|                                                                   |   csql> SELECT * FROM tbl;                                                        |
|                                                                   |                                                                                   |
|                                                                   |   There are no results.                                                           |
|                                                                   |                                                                                   |
|                                                                   |   -- commit to start a new transaction with a new snapshot                        |
|                                                                   |   csql> COMMIT WORK;                                                              |
|                                                                   |                                                                                   |
|                                                                   |   -- the new snapshot should see committed row                                    |
|                                                                   |   csql> SELECT * FROM tbl;                                                        |
|                                                                   |                                                                                   |
|                                                                   |       host_year  nation_code                                                      |
|                                                                   |   ===================================                                             |
|                                                                   |            2008  'AUS'                                                            |
|                                                                   |                                                                                   |
+-------------------------------------------------------------------+-----------------------------------------------------------------------------------+
**Example 2: Deleting a row**

+-------------------------------------------------------------------+-----------------------------------------------------------------------------------+
| session 1                                                         | session 2                                                                         |
+===================================================================+===================================================================================+
| .. code-block:: sql                                               | .. code-block:: sql                                                               |
|                                                                   |                                                                                   |
|   csql> ;autocommit off                                           |   csql> ;autocommit off                                                           |
|                                                                   |                                                                                   |
|   AUTOCOMMIT IS OFF                                               |   AUTOCOMMIT IS OFF                                                               |
|                                                                   |                                                                                   |
|   csql> set transaction isolation level REPEATABLE READ;          |   csql> set transaction isolation level REPEATABLE READ;                          |
|                                                                   |                                                                                   |
|   Isolation level set to:                                         |   Isolation level set to:                                                         |
|   REPEATABLE READ                                                 |   REPEATABLE READ                                                                 |
|                                                                   |                                                                                   |
+-------------------------------------------------------------------+-----------------------------------------------------------------------------------+
| .. code-block:: sql                                               |                                                                                   |
|                                                                   |                                                                                   |
|   csql> CREATE TABLE tbl(host_year integer, nation_code char(3)); |                                                                                   |
|   csql> INSERT INTO tbl VALUES (2008, 'AUS');                     |                                                                                   |
|   csql> COMMIT WORK;                                              |                                                                                   |
|                                                                   |                                                                                   |
+-------------------------------------------------------------------+-----------------------------------------------------------------------------------+
| .. code-block:: sql                                               |                                                                                   |
|                                                                   |                                                                                   |
|   -- delete the row without committing                            |                                                                                   |
|   csql> DELETE FROM tbl WHERE nation_code = 'AUS';                |                                                                                   |
|                                                                   |                                                                                   |
|   -- this transaction sees its own changes                        |                                                                                   |
|   csql> SELECT * FROM tbl;                                        |                                                                                   |
|                                                                   |                                                                                   |
|   There are no results.                                           |                                                                                   |
|                                                                   |                                                                                   |
+-------------------------------------------------------------------+-----------------------------------------------------------------------------------+
|                                                                   | .. code-block:: sql                                                               |
|                                                                   |                                                                                   |
|                                                                   |   -- delete was not committed, so the row is visible to this snapshot             |
|                                                                   |   csql> SELECT * FROM tbl;                                                        |
|                                                                   |                                                                                   |
|                                                                   |       host_year  nation_code                                                      |
|                                                                   |   ===================================                                             |
|                                                                   |            2008  'AUS'                                                            |
|                                                                   |                                                                                   |
+-------------------------------------------------------------------+-----------------------------------------------------------------------------------+
| .. code-block:: sql                                               |                                                                                   |
|                                                                   |                                                                                   |
|   csql> COMMIT WORK;                                              |                                                                                   |
|                                                                   |                                                                                   |
+-------------------------------------------------------------------+-----------------------------------------------------------------------------------+
|                                                                   | .. code-block:: sql                                                               |
|                                                                   |                                                                                   |
|                                                                   |   -- delete was committed, but the row is still visible to this snapshot          |
|                                                                   |   csql> SELECT * FROM tbl;                                                        |
|                                                                   |                                                                                   |
|                                                                   |       host_year  nation_code                                                      |
|                                                                   |   ===================================                                             |
|                                                                   |            2008  'AUS'                                                            |
|                                                                   |                                                                                   |
|                                                                   |   -- commit to start a new transaction with a new snapshot                        |
|                                                                   |   csql> COMMIT WORK;                                                              |
|                                                                   |                                                                                   |
|                                                                   |   -- the new snapshot can no longer see deleted row                               |
|                                                                   |   csql> SELECT * FROM tbl;                                                        |
|                                                                   |                                                                                   |
|                                                                   |   There are no results.                                                           |
|                                                                   |                                                                                   |
+-------------------------------------------------------------------+-----------------------------------------------------------------------------------+

**Example 3: Updating a row**

+-------------------------------------------------------------------+-----------------------------------------------------------------------------------+
| session 1                                                         | session 2                                                                         |
+===================================================================+===================================================================================+
| .. code-block:: sql                                               | .. code-block:: sql                                                               |
|                                                                   |                                                                                   |
|   csql> ;autocommit off                                           |   csql> ;autocommit off                                                           |
|                                                                   |                                                                                   |
|   AUTOCOMMIT IS OFF                                               |   AUTOCOMMIT IS OFF                                                               |
|                                                                   |                                                                                   |
|   csql> set transaction isolation level REPEATABLE READ;          |   csql> set transaction isolation level REPEATABLE READ;                          |
|                                                                   |                                                                                   |
|   Isolation level set to:                                         |   Isolation level set to:                                                         |
|   REPEATABLE READ                                                 |   REPEATABLE READ                                                                 |
|                                                                   |                                                                                   |
+-------------------------------------------------------------------+-----------------------------------------------------------------------------------+
| .. code-block:: sql                                               |                                                                                   |
|                                                                   |                                                                                   |
|   csql> CREATE TABLE tbl(host_year integer, nation_code char(3)); |                                                                                   |
|   csql> INSERT INTO tbl VALUES (2008, 'AUS');                     |                                                                                   |
|   csql> COMMIT WORK;                                              |                                                                                   |
|                                                                   |                                                                                   |
+-------------------------------------------------------------------+-----------------------------------------------------------------------------------+
| .. code-block:: sql                                               |                                                                                   |
|                                                                   |                                                                                   |
|   -- delete the row without committing                            |                                                                                   |
|   csql> UPDATE tbl SET host_year = 2012 WHERE nation_code = 'AUS';|                                                                                   |
|                                                                   |                                                                                   |
|   -- this transaction sees new version, host_year = 2012          |                                                                                   |
|   csql> SELECT * FROM tbl;                                        |                                                                                   |
|                                                                   |                                                                                   |
|       host_year  nation_code                                      |                                                                                   |
|   ===================================                             |                                                                                   |
|            2012  'AUS'                                            |                                                                                   |
|                                                                   |                                                                                   |
+-------------------------------------------------------------------+-----------------------------------------------------------------------------------+
|                                                                   | .. code-block:: sql                                                               |
|                                                                   |                                                                                   |
|                                                                   |   -- update was not committed, so this snapshot sees old version                  |
|                                                                   |   csql> SELECT * FROM tbl;                                                        |
|                                                                   |                                                                                   |
|                                                                   |       host_year  nation_code                                                      |
|                                                                   |   ===================================                                             |
|                                                                   |            2008  'AUS'                                                            |
|                                                                   |                                                                                   |
+-------------------------------------------------------------------+-----------------------------------------------------------------------------------+
| .. code-block:: sql                                               |                                                                                   |
|                                                                   |                                                                                   |
|   csql> COMMIT WORK;                                              |                                                                                   |
|                                                                   |                                                                                   |
+-------------------------------------------------------------------+-----------------------------------------------------------------------------------+
|                                                                   | .. code-block:: sql                                                               |
|                                                                   |                                                                                   |
|                                                                   |   -- update was committed, but this snapshot still sees old version               |
|                                                                   |   csql> SELECT * FROM tbl;                                                        |
|                                                                   |                                                                                   |
|                                                                   |       host_year  nation_code                                                      |
|                                                                   |   ===================================                                             |
|                                                                   |            2008  'AUS'                                                            |
|                                                                   |                                                                                   |
|                                                                   |   -- commit to start a new transaction with a new snapshot                        |
|                                                                   |   csql> COMMIT WORK;                                                              |
|                                                                   |                                                                                   |
|                                                                   |   -- the new snapshot can see new version, host_year = 2012                       |
|                                                                   |   csql> SELECT * FROM tbl;                                                        |
|                                                                   |                                                                                   |
|                                                                   |       host_year  nation_code                                                      |
|                                                                   |   ===================================                                             |
|                                                                   |            2012  'AUS'                                                            |
|                                                                   |                                                                                   |
+-------------------------------------------------------------------+-----------------------------------------------------------------------------------+
**Example 4: Different versions can be visible to different transactions**

+-------------------------------------------------------------------+----------------------------------------+----------------------------------------+
| session 1                                                         | session 2                              | session 3                              |
+===================================================================+========================================+========================================+
| .. code-block:: sql                                               |                                        |                                        |
|                                                                   |                                        |                                        |
|   csql> INSERT INTO tbl VALUES (2008, 'AUS');                     |                                        |                                        |
|   csql> COMMIT WORK;                                              |                                        |                                        |
|                                                                   |                                        |                                        |
+-------------------------------------------------------------------+----------------------------------------+----------------------------------------+
| .. code-block:: sql                                               | .. code-block:: sql                    |                                        |
|                                                                   |                                        |                                        |
|   -- update row                                                   |                                        |                                        |
|   csql> UPDATE tbl SET host_year = 2012 WHERE nation_code = 'AUS';|                                        |                                        |
|                                                                   |                                        |                                        |
|   csql> SELECT * FROM tbl;                                        |   csql> SELECT * FROM tbl;             |                                        |
|                                                                   |                                        |                                        |
|       host_year  nation_code                                      |       host_year  nation_code           |                                        |
|   ===================================                             |   ===================================  |                                        |
|            2012  'AUS'                                            |            2008  'AUS'                 |                                        |
|                                                                   |                                        |                                        |
+-------------------------------------------------------------------+----------------------------------------+----------------------------------------+
| .. code-block:: sql                                               |                                        |                                        |
|                                                                   |                                        |                                        |
|   csql> COMMIT WORK;                                              |                                        |                                        |
|                                                                   |                                        |                                        |
+-------------------------------------------------------------------+----------------------------------------+----------------------------------------+
| .. code-block:: sql                                               |  .. code-block:: sql                   |  .. code-block:: sql                   |
|                                                                   |                                        |                                        |
|   csql> UPDATE tbl SET host_year = 2016 WHERE nation_code = 'AUS';|                                        |                                        |
|                                                                   |                                        |                                        |
|   csql> SELECT * FROM tbl;                                        |   csql> SELECT * FROM tbl;             |   csql> SELECT * FROM tbl;             |
|                                                                   |                                        |                                        |
|       host_year  nation_code                                      |       host_year  nation_code           |       host_year  nation_code           |
|   ===================================                             |   ===================================  |   ===================================  |
|            2016  'AUS'                                            |            2008  'AUS'                 |            2012  'AUS'                 |
|                                                                   |                                        |                                        |
+-------------------------------------------------------------------+----------------------------------------+----------------------------------------+

VACUUM
------

Creating new versions for each update and keeping old versions on delete could lead to unlimited database size growth, definitely a major issue for a database. Therefore, a clean up system is necessary, to remove obsolete data and reclaim the occupied space for reuse.

Each row version goes through same stages:

  1. newly inserted, not committed, visible only to its inserter.
  2. committed, invisible to preceding transactions, visible to future transactions.
  3. deleted, not committed, visible to other transactions, invisible to the deleter.
  4. committed, still visible to preceding transactions, invisible to future transactions.
  5. invisible to all active transactions.
  6. removed from database.

The role of the clean up system is to get versions from stage 5 to 6. This system is called **VACUUM** in CUBRID.

**VACUUM** system was developed with the guidance of three principles:

*   **VACUUM** must be correct and complete. **VACUUM** should never remove data still visible to some and it should not miss any obsolete data.
*   **VACUUM** must be discreet. Since clean-up process changes database content, there may be some interference in the activity of live transactions, but it must be kept to the minimum possible.
*   **VACUUM** must be fast and efficient. If **VACUUM** is too slow and if it starts lagging, the database state can deteriorate, thus the overall performance can be affected.

With these principles in mind, **VACUUM** implementation uses existing recovery logging, because:

*   The address is kept among recovery data for both heap and index changes. This allows **VACUUM** go directly to target, rather than scanning the database.
*   Processing log data rarely interferes with the work of active workers.

Log recovery was adapted to **VACUUM** needs by adding MVCCID information to logged data. **VACUUM** can decide based on MVCCID if the log entry is ready to be processed. MVCCID's that are still visible to active transactions cannot be processed. In due time, each MVCCID becomes old enough and all changes using the MVCCID become invisible.

Each transaction keeps the oldest MVCCID it considers active. The oldest MVCCID considered active by all running transactions is determined by the smallest oldest MVCCID of all transactions. Anything below this value is invisible and **VACUUM** can clean.

VACUUM Parallel Execution
+++++++++++++++++++++++++

According to the third principle of **VACUUM** it must be fast and it should not fall behind active workers. It is obvious that one thread cannot handle all the **VACUUM** works if system workload is heavy, thus it had to be parallelized.

To achieve parallelization, the log data was split into fixed size blocks. Each block generates one vacuum job, when the time is right (the most recent MVCCID can be vacuumed, which means all logged operations in the block can be vacuumed). Vacuum jobs are picked up by multiple **VACUUM Workers** that clean the database based on relevant log entries found in the log block. The tracking of log blocks and generating vacuum jobs is done by the **VACUUM Master**.

VACUUM Data
+++++++++++

Aggregated data on log blocks is stored in vacuum data file. Since the vacuum job generated by an operations occurs later in time, the data must be saved until the job can be executed, and it must also be persistent even if the server crashes. No operation is allowed to leak and not be vacuumed. If the server crashes, some jobs may be executed twice, which is preferable to not being executed at all.

After a job has been successfully executed, the aggregated data on the processed log block is removed.

Aggregated log block data is not added directly to vacuum data. A latch-free buffer is used to avoid synchronizing active working threads (which generate the log blocks and their aggregated data) with the vacuum system. **VACUUM Master** wakes up periodically, dumps everything in buffer to vacuum data, removes data already process and generates new jobs (if available).

VACUUM jobs
+++++++++++

Vacuum job execution steps:

  1. **Log pre-fetch**. Vacuum master or workers pre-fetch log pages to be processed by the job.
  2. **Repeat for each log record**:

    1. **Read** log record.
    2. **Check dropped file.** If the log record points to dropped file, proceed to next log record.
    3. **Execute index vacuum and collect heap OID's**

      * If log record belongs to index, execute vacuum immediately.
      * If log record belongs to heap, collect OID to be vacuumed later.

  3. **Execute heap vacuum** based on collected OID's.
  4. **Complete job.** Mark the job as completed in vacuum data.

Several measures were taken to ease log page reading and to optimize vacuum execution.

Tracking dropped files
++++++++++++++++++++++

When a transaction drops a table or an index, it usually locks the affected table(s) and prevents others from accessing it. Opposed to active workers, **VACUUM** Workers are not allowed to use locking system, for two reasons: interference with active workers must be kept to the minimum, and **VACUUM** system is never supposed to stop as long as it has data to clean. Moreover, **VACUUM** is not allowed to skip any data that needs cleaning. This has two consequences:

  1. **VACUUM** doesn't stop from cleaning a file belonging to a dropped table or a dropped index until the dropper commits. Even if a transaction drops a table, its file is not immediately destroyed and it can still be accessed. The actual destruction is postponed until after commit.
  2. Before the actual file destruction, **VACUUM** system must be notified. The dropper sends a notification to **VACUUM** system and then waits for the confirmation. **VACUUM** works on very short iterations and it checks for new dropped files frequently, so the dropper doesn't have to wait for a long time.

After a file is dropped, **VACUUM** will ignore all found log entries that belong to the file. The file identifier, paired with an MVCCID that marks the moment of drop, is stored in a persistent file until **VACUUM** decides it is safe to remove it (the decision is based on the smallest MVCCID not yet vacuumed).

.. _lock-protocol:

잠금 프로토콜
=============

CUBRID는 동시성 제어를 위해 2단계 잠금 프로토콜(2-phase locking protocol, 2PL)을 사용하여 트랜잭션 스케줄을 관리한다. 이는 트랜잭션이 사용하는 자원, 즉 객체에 대해 상호 배제 기능을 제공하는 기법이다. 확장 단계(growing phase)에서는 트랜잭션들이 잠금 연산만 수행할 수 있고 잠금 해제(unlock) 연산은 수행할 수 없다. 축소 단계(shrinking phase)에서는 트랜잭션들이 잠금 해제(unlock) 연산만 수행할 수 있고 잠금 연산은 수행할 수 없다. 즉, 트랜잭션 T1이 특정 객체에 대해 읽기 또는 갱신 연산을 수행하기 전에 반드시 잠금 연산을 먼저 수행하고, T1을 종료하기 전에 잠금 해제 연산을 수행해야 한다.

잠금의 단위
-----------

CUBRID는 잠금의 개수를 줄이기 위해서 단위 잠금(granularity locking) 프로토콜을 사용한다. 단위 잠금 프로토콜에서는 잠금 단위의 크기에 따라 계층으로 모델화되며, 행 잠금(row lock), 테이블 잠금(table lock), 데이터베이스 잠금(database lock)이 있다. 이때, 단위가 큰 잠금은 작은 단위의 잠금을 내포한다.

잠금을 설정하고 해제하는 과정에서 발생하는 성능 손실을 잠금 비용(overhead)이라고 하는데, 큰 단위보다 작은 단위의 잠금을 수행할 때 이러한 잠금 비용이 높아지고 대신 트랜잭션 동시성은 향상된다. 따라서, CUBRID는 잠금 비용과 트랜잭션 동시성을 고려하여 잠금 단위를 결정한다. 예를 들어, 한 트랜잭션이 테이블의 모든 행들을 조회하는 경우 행 단위로 잠금을 설정/해제하는 비용이 너무 높으므로 차라리 해당 테이블에 잠금을 설정한다. 이처럼 테이블에 잠금이 설정되면 트랜잭션 동시성이 저하되므로, 동시성을 보장하려면 풀 스캔(full scan)이 발생하지 않도록 적절한 인덱스를 사용해야 할 것이다.

이와 같은 잠금 관리를 위해 CUBRID는 잠금 에스컬레이션(lock escalation) 기법을 사용하여 설정 가능한 단위 잠금의 수를 제한한다. 예를 들어, 한 트랜잭션이 행 단위에서 특정 개수 이상의 잠금을 가지고 있으면 시스템은 계층적으로 상위 단위인 테이블에 대해 잠금을 요청하기 시작한다. 단, 상위 단위로 잠금 에스컬레이션을 수행하기 위해서는 어떤 트랜잭션도 상위 단위 객체에 대한 잠금을 가지고 있지 않아야 한다. 그래야만 잠금 변환에 따른 교착 상태(deadlock)를 예방할 수 있다. 이때, 작은 단위에서 허용하는 잠금 개수는 시스템 파라미터 **lock_escalation** 을 통해 설정할 수 있다.

.. _lock-mode:

잠금 모드의 종류와 호환성
-------------------------

CUBRID는 트랜잭션이 수행하고자 하는 연산의 종류에 따라 획득하고자 하는 잠금 모드를 결정하며, 다른 트랜잭션에 의해 이미 선점된 잠금 모드의 종류에 따라 잠금 공유 여부를 결정한다. 이와 같은 잠금에 대한 결정은 시스템이 자동으로 수행하며, 사용자에 의한 수동 지정은 허용되지 않는다. CUBRID의 잠금 정보를 확인하기 위해서는 **cubrid lockdb** *db_name* 명령어를 사용하며, 자세한 내용은 :ref:`lockdb` 을 참고한다.

*   **공유 잠금(shared lock, S_LOCK)**

    객체에 대해 읽기 연산을 수행하기 전에 획득하며, 여러 트랜잭션이 동일 객체에 대해 획득할 수 있는 잠금이다.

    트랜잭션 T1이 특정 객체에 대해 읽기 연산을 수행하기 전에 공유 잠금을 먼저 획득한다. 이때, 트랜잭션 T2, T3은 동시에 그 객체에 대해 읽기 연산을 수행할 수 있으나 갱신 연산을 수행할 수 없다.
    
    .. note::

        *   격리 수준이 READ COMMITTED(4)인 경우 트랜잭션 T1이 커밋되기 전이라도 읽기 연산을 완료하면 획득한 공유 잠금을 즉시 해제하므로, 다른 트랜잭션 중 하나가 해당 객체에 대한 갱신 또는 삭제 연산을 수행할 수 있다.
        *   격리 수준이 REPEATABLE READ(5)인 경우, 트랜잭션 T1이 커밋될 때까지 공유 잠금을 유지하므로, 다른 트랜잭션 중 하나가 해당 객체에 대한 갱신 또는 삭제 연산을 수행할 수 없다.

*   **배타 잠금(exclusive lock, X_LOCK)**

    객체에 대해 갱신 연산을 수행하기 전에 획득하며, 하나의 트랜잭션만 획득할 수 있는 잠금이다.

    트랜잭션 T1이 특정 객체 X에 대해 갱신 연산을 수행하기 전에 배타 잠금을 먼저 획득하고, 갱신 연산을 완료하더라도 트랜잭션 T1이 커밋될 때까지 배타 잠금을 해제하지 않는다. 따라서, 트랜잭션 T2, T3은 트랜잭션 T1이 배타 잠금을 해제하기 전까지는 X에 대한 읽기 연산도 수행할 수 없다.

*   **갱신 잠금(update lock, U_LOCK)** 

    갱신 연산을 수행하기 전, 조건절에서 읽기 연산을 수행할 때 획득하는 잠금이다. 즉, UPDATE ... WHERE 문 또는 DELETE ... WHERE 문에서 사용된다.

    예를 들어 **WHERE** 절과 결합된 **UPDATE** 문을 수행하는 경우, **WHERE** 절에서 인덱스 검색을 하거나 풀 스캔 검색을 수행할 때 행 단위로 갱신 잠금을 획득하고, 조건을 만족하는 결과 행들에 대해서만 배타 잠금을 획득하여 갱신 연산을 수행한다. 이처럼 갱신 잠금은 실제 갱신 연산을 수행할 때 배타 잠금으로 변환되며, 이는 다른 트랜잭션이 동일한 객체에 대해 읽기 연산을 수행하지 못하도록 하므로 준 배타 잠금이라고 할 수 있다.

*   **의도 잠금(내재된 잠금, intent lock)**

    특정 단위의 객체에 걸리는 잠금을 보호하기 위하여 이 객체보다 상위 단위의 객체에 내재적으로 설정하는 잠금을 의미한다.

    예를 들어, 특정 행에 공유 잠금이 요청되면 이보다 계층적으로 상위에 있는 테이블에도 의도 공유 잠금을 함께 설정하여 다른 트랜잭션에 의해 테이블이 잠금되는 것을 예방한다. 따라서, 의도 잠금은 계층적으로 가장 낮은 단위인 행에 대해서는 설정되지 않으며, 이보다 높은 단위의 객체에 대해서만 설정된다. 의도 잠금의 종류는 다음과 같다.

    *   **의도 공유 잠금(intent shared lock, IS_LOCK)**

        특정 행에 공유 잠금이 설정됨에 따라 상위 객체인 테이블에 의도 공유 잠금이 설정되면, 다른 트랜잭션은 칼럼을 추가하거나 테이블 이름을 변경하는 등의 테이블 스키마를 변경할 수 없고, 모든 행을 갱신하는 작업을 수행할 수 없다. 그러나 일부 행을 갱신하는 작업이나, 모든 행을 조회하는 작업은 허용된다.

    *   **의도 배타 잠금(intent exclusive lock, IX_LOCK)** 
    
        특정 행에 배타 잠금이 설정됨에 따라 상위 객체인 테이블에 의도 배타 잠금이 설정되면, 다른 트랜잭션은 테이블 스키마를 변경할 수 없고, 모든 행을 갱신하는 작업은 물론, 모든 행을 조회하는 작업은 수행할 수 없다. 그러나, 일부 행을 갱신하는 작업은 허용된다.

    *   **공유 의도 배타 잠금(shared with intent exclusive lock, SIX_LOCK)** 
    
        계층적으로 더 낮은 모든 객체에 설정된 공유 잠금을 보호하고, 계층적으로 더 낮은 일부 객체에 대한 의도 배타 잠금을 보호하기 위하여 상위 객체에 내재적으로 설정되는 잠금이다.

        테이블에 공유 의도 배타 잠금이 설정되면, 다른 트랜잭션은 테이블 스키마를 변경할 수 없고, 모든 행/일부 행을 갱신할 수 없으며, 모든 행을 조회할 수 없다. 그러나, 일부 행을 조회하는 작업은 허용된다.

*   **키 잠금**

    키가 존재하는 행에 대해 SELECT, INSERT, UPDATE, DELETE 등의 작업을 수행할 때 키에 잠금을 획득한다.

    예를 들어 어떤 값을 INSERT하면, 해당 값에 대해 X_LOCK을 획득하고 해당 키와 다음 키에 NS_LOCK을 획득한다.

    어떤 값을 UPDATE/DELETE하면, 지정한 범위에 해당하는 모든 키와 범위 내 가장 마지막 키의 다음 키에 NX_LOCK을 획득한다.

    *   **다음 키 공유 잠금(next-key shared lock, NS_LOCK)**

        고유 키가 존재하는 행에 대해 INSERT 작업을 수행할 때 해당 작업이 영향을 주는 범위를 보호하기 위해 다음 키에 잠금을 획득한다.
    
    *   **다음 키 배타 잠금(next-key exclusive lock, NX)**
    
        고유 키가 존재하는 행에 대해 UPDATE, DELETE 작업을 수행할 때 해당 작업이 영향을 주는 범위를 보호하기 위해 해당 범위 앞의 키와 다음 키에 잠금을 획득한다.
    
*   **스키마 잠금**
    
    DDL 작업을 수행할 때 스키마 잠금을 획득한다.
    
    *   **스키마 안정 잠금(schema stability lock, SCH_S_LOCK)**

        질의 컴파일을 수행하는 동안 획득되며 질의에 포함된 스키마가 다른 트랜잭션에 의해 수정되지 않음을 보장한다. 

    *   **스키마 수정 잠금(schema modification lock, SCH_M_LOCK)**

        DDL(ALTER/CREATE/DROP)을 실행하는 동안 획득되며 다른 트랜잭션이 수정된 스키마에 접근하는 것을 방지한다.

    ALTER, CREATE INDEX 등 일부 DDL 연산은 SCH_M_LOCK을 직접 획득하지 않는다. 예를 들어 필터링된 인덱스를 생성할 때, CUBRID는 필터링 표현식에 대한 타입 검사를 수행한다. 이 기간 동안, 대상 테이블에 유지되는 잠금은 다른 타입 검사 연산의 경우처럼 SCH-S이다. 이러한 방식은 DDL 연산이 컴파일되는 동안 다른 트랜잭션이 연산을 수행하는 것을 허용하여, 동시성을 높일 수 있다는 이점이 있다.

    하지만 이 방식은 같은 테이블에 동시에 DDL 연산을 수행할 때 교착 상태를 회피할 수 없다는 단점 또한 존재한다. SCH-S 잠금으로 인한 교착 상태의 예는 다음과 같다.

    +---------------------------------------------------------+---------------------------------------------------------+
    | T1                                                      | T2                                                      |
    +=========================================================+=========================================================+
    | ::                                                      | ::                                                      |
    |                                                         |                                                         |
    |  CREATE INDEX i_t_i on t( i ) WHERE i > 0               |   CREATE INDEX i_t_j on t(j) WHERE j > 0                |
    +---------------------------------------------------------+---------------------------------------------------------+
    | "i > 0" 조건의 타입 검사를 수행하는 동안 SCH-S 잠금     |                                                         |
    +---------------------------------------------------------+---------------------------------------------------------+
    |                                                         | "j > 0" 조건의 타입 검사를 수행하는 동안 SCH-S 잠금     |
    +---------------------------------------------------------+---------------------------------------------------------+
    | SCH-M 잠금을 요청하나 T2의 SCH-S 잠금때문에 대기 상태   |                                                         |
    +---------------------------------------------------------+---------------------------------------------------------+
    |                                                         | SCH-M 잠금을 요청하나 T1의 SCH-S 잠금때문에 대기 상태   |
    +---------------------------------------------------------+---------------------------------------------------------+
    
.. note:: 잠금에 대해 요약하면 다음과 같다.

    *   잠금 대상 객체에는 행(instance), 키(key), 스키마(class)가 있다. 잠금 대상 객체를 기준으로 잠금의 종류를 나누면 다음과 같다.

        *   행 잠금: S_LOCK, X_LOCK, U_LOCK
        
        *   키 잠금: NS_LOCK, NX_LOCK
        
        *   스키마 잠금: IX_LOCK, IS_LOCK, SIX_LOCK, SCH_S_LOCK, SCH_M_LOCK
        
    *   행 잠금과 스키마 잠금은 서로에게 영향을 끼친다.
        
    *   키 잠금은 행 잠금, 스키마 잠금과 무관하다.

위에서 설명한 잠금들의 호환 관계(lock compatibility)를 정리하면 아래의 표와 같다. 호환된다는 것은 잠금 보유자(lock holder)가 특정 객체에 대해 획득한 잠금과 중복하여 잠금 요청자(lock requester)가 잠금을 획득할 수 있다는 의미이다.

**잠금 호환성**

(O: TRUE, X: FALSE, -: N/A)

+----------------------------------+-----------------------------------------------------------------------------------------------------------------------------------+
|                                  | **잠금 보유자(Lock holder)**                                                                                                      |
|                                  +-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+
|                                  | **NULL**  | **SCH-S** | **IS**    | **S**     | **IX**    | **SIX**   | **U**     | **X**     | **NS**    | **NX**    | **SCH-M** |
+----------------------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+
| **잠금 요청자**      | **NULL**  | O         | O         | O         | O         | O         | O         | O         | O         | O         | O         | O         |
| **(Lock requester)** |           |           |           |           |           |           |           |           |           |           |           |           |
|                      +-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+
|                      | **SCH-S** | O         | O         | O         | O         | O         | O         | \-        | O         | \-        | \-        | O         |
|                      +-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+
|                      | **IS**    | O         | O         | O         | O         | O         | O         | \-        | X         | \-        | \-        | X         |
|                      +-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+
|                      | **S**     | O         | O         | O         | O         | X         | X         | X         | X         | X         | X         | X         |
|                      +-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+
|                      | **IX**    | O         | O         | O         | X         | O         | X         | \-        | X         | \-        | \-        | X         |
|                      +-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+
|                      | **SIX**   | O         | O         | O         | X         | X         | X         | \-        | X         | \-        | \-        | X         |
|                      +-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+
|                      | **U**     | O         | \-        | \-        | O         | \-        | \-        | X         | X         | \-        | \-        | \-        |
|                      +-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+
|                      | **X**     | O         | O         | X         | X         | X         | X         | X         | X         | \-        | \-        | X         |
|                      +-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+
|                      | **NS**    | O         | \-        | \-        | X         | \-        | \-        | \-        | \-        | O         | X         | \-        |
|                      +-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+
|                      | **NX**    | O         | \-        | \-        | X         | \-        | \-        | \-        | \-        | X         | X         | \-        |
|                      +-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+
|                      | **SCH-M** | O         | X         | X         | X         | X         | X         | \-        | X         | \-        | \-        | X         |
+----------------------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+

*   **NULL**\: 아무 잠금도 없는 상태

**잠금 변환 테이블**

+----------------------------------+-----------------------------------------------------------------------------------------------------------------------------------+
|                                  | **획득 잠금 모드(Granted lock mode)**                                                                                             |
|                                  +-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+
|                                  | **NULL**  | **SCH-S** | **IS**    | **S**     | **IX**    | **SIX**   | **U**     | **X**     | **NS**    | **NX**    | **SCH-M** |
+----------------------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+
| **요청 잠금 모드**   | **NULL**  | NULL      | SCH-S     | IS        | S         | IX        | SIX       | U         | X         | NS        | NX        | SCH-M     |
|                      +-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+
|                      | **SCH-S** | SCH-S     | SCH-S     | IS        | S         | IX        | SIX       | \-        | X         | \-        | \-        | SCH-M     |
|                      +-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+
|                      | **IS**    | IS        | IS        | IS        | S         | IX        | SIX       | \-        | X         | \-        | \-        | SCH-M     |
|                      +-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+
|                      | **S**     | S         | S         | S         | S         | SIX       | SIX       | U         | X         | NX        | NX        | SCH-M     |
|                      +-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+
|                      | **IX**    | IX        | IX        | IX        | SIX       | IX        | SIX       | \-        | X         | \-        | \-        | SCH-M     |
|                      +-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+
|                      | **SIX**   | SIX       | SIX       | SIX       | SIX       | SIX       | SIX       | \-        | X         | \-        | \-        | SCH-M     |
|                      +-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+
|                      | **U**     | U         | \-        | \-        | U         | \-        | \-        | U         | X         | \-        | \-        | \-        |
|                      +-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+
|                      | **X**     | X         | X         | X         | X         | X         | X         | X         | X         | \-        | \-        | SCH-M     |
|                      +-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+
|                      | **NS**    | NS        | \-        | \-        | NX        | \-        | \-        | \-        | \-        | NS        | NX        | \-        |
|                      +-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+
|                      | **NX**    | NX        | \-        | \-        | NX        | \-        | \-        | \-        | \-        | NX        | NX        | \-        |
|                      +-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+
|                      | **SCH-M** | SCH-M     | SCH-M     | SCH-M     | SCH-M     | SCH-M     | SCH-M     | \-        | SCH-M     | \-        | \-        | SCH-M     |
+----------------------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+

다음은 격리 수준(isolation level)을 4로 설정한 경우 INSERT한 데이터가 행에 X_LOCK, 키에 대해 NS_LOCK을 설정하는 예이다.

+---------------------------------------------------------+---------------------------------------------------------+----------------------------------------------------------------------------+
| T1                                                      | T2                                                      | 설명                                                                       |
+=========================================================+=========================================================+============================================================================+
| ::                                                      | ::                                                      | AUTOCOMMIT OFF, READ COMMITTED로 설정                                      |
|                                                         |                                                         |                                                                            |
|   csql> ;au off                                         |   csql> ;au off                                         |                                                                            |
|   SET TRANSACTION ISOLATION LEVEL 4;                    |   SET TRANSACTION ISOLATION LEVEL 4;                    |                                                                            |
+---------------------------------------------------------+---------------------------------------------------------+----------------------------------------------------------------------------+
| ::                                                      |                                                         |                                                                            |
|                                                         |                                                         |                                                                            |
|   CREATE TABLE tbl(a INT PRIMARY KEY, b INT);           |                                                         |                                                                            |
|   INSERT INTO tbl                                       |                                                         |                                                                            |
|     VALUES (10, 10), (30, 30), (50, 50), (70, 70);      |                                                         |                                                                            |
|   COMMIT;                                               |                                                         |                                                                            |
+---------------------------------------------------------+---------------------------------------------------------+----------------------------------------------------------------------------+
| ::                                                      |                                                         | 행 20에 X_LOCK, 키 30에 NS_LOCK을 획득한다.                                |
|                                                         |                                                         |                                                                            |
|   INSERT INTO tbl VALUES (20, 20);                      |                                                         |                                                                            |
+---------------------------------------------------------+---------------------------------------------------------+----------------------------------------------------------------------------+
|                                                         | ::                                                      | 대기. 행 20에 X_LOCK이 잠겨있으므로 T2의 S_LOCK을 허용하지 않는다.         |
|                                                         |                                                         |                                                                            |
|                                                         |   SELECT * FROM tbl WHERE a <= 20;                      |                                                                            |
+---------------------------------------------------------+---------------------------------------------------------+----------------------------------------------------------------------------+
|                                                         | 인터럽트(ctrl-C)로 작업 취소                            |                                                                            |
+---------------------------------------------------------+---------------------------------------------------------+----------------------------------------------------------------------------+
|                                                         | ::                                                      | 허용. T2의 25보다 큰 행에 대한 S_LOCK을 요청하므로 키 잠금과는 무관하다.   |
|                                                         |                                                         |                                                                            |
|                                                         |   SELECT * FROM tbl WHERE a > 25;                       |                                                                            |
+---------------------------------------------------------+---------------------------------------------------------+----------------------------------------------------------------------------+
|                                                         | ::                                                      | 허용. 키 30에 NS_LOCK이 잠겨있지만 T2는 행에 대해 U_LOCK 획득 후           |
|                                                         |                                                         | X_LOCK으로 변환하는데, 행 잠금은 T1의 키 잠금에 영향을 주지는 않는다.      |
|                                                         |   UPDATE tbl SET b=100 WHERE a > 25 AND a < 40;         | 또한 T2는 키 30,키 50에 대한 NX_LOCK을 획득한다.                           |
+---------------------------------------------------------+---------------------------------------------------------+----------------------------------------------------------------------------+
| ::                                                      |                                                         | T1의 잠금이 해제된다.                                                      |
|                                                         |                                                         |                                                                            |
|   COMMIT;                                               |                                                         |                                                                            |
+---------------------------------------------------------+---------------------------------------------------------+----------------------------------------------------------------------------+
| ::                                                      |                                                         | 대기. T2가 키 30, 키 50에 NX_LOCK을 보유하고 있는데 행 22를                |
|                                                         |                                                         | INSERT하려면 T1이 키 30에 대한 NS_LOCK을 획득해야 하므로 대기하게 된다.    |
|   INSERT INTO tbl VALUES (22, 22);                      |                                                         |                                                                            |
+---------------------------------------------------------+---------------------------------------------------------+----------------------------------------------------------------------------+

다음은 격리 수준(isolation level)을 4로 설정한 경우 T2가 업데이트한 데이터를 커밋할 때까지 T1이 대기하는 예이다.

+-------------------------------------------------------------------------------+-------------------------------------------------------------------------------+
| T1                                                                            | T2                                                                            |
+===============================================================================+===============================================================================+
| ::                                                                            | ::                                                                            |
|                                                                               |                                                                               |
|   csql> ;autocommit off                                                       |   csql> ;autocommit off                                                       |
|                                                                               |                                                                               |
|   AUTOCOMMIT IS OFF                                                           |   AUTOCOMMIT IS OFF                                                           |
|                                                                               |                                                                               |
|   csql> SET TRANSACTION ISOLATION LEVEL 4;                                    |   csql> SET TRANSACTION ISOLATION LEVEL 4;                                    |
|                                                                               |                                                                               |
|   Isolation level set to:                                                     |   Isolation level set to:                                                     |
|   READ COMMITTED                                                              |   READ COMMITTED                                                              |
|                                                                               |                                                                               |
|                                                                               | ::                                                                            |
|                                                                               |                                                                               |
|                                                                               |   $ cubrid lockdb demodb                                                      |
|                                                                               |                                                                               |
|                                                                               |   *** Lock Table Dump ***                                                     |
|                                                                               |                                                                               |
|                                                                               |   ...                                                                         |
|                                                                               |                                                                               |
|                                                                               |   Object Lock Table:                                                          |
|                                                                               |         Current number of objects which are locked    = 0                     |
|                                                                               |         Maximum number of objects which can be locked = 10000                 |
|                                                                               |                                                                               |
|                                                                               |   ...                                                                         |
+-------------------------------------------------------------------------------+-------------------------------------------------------------------------------+
| ::                                                                            |                                                                               |
|                                                                               |                                                                               |
|   csql> SELECT nation_code, gold FROM participant WHERE nation_code='USA';    |                                                                               |
|                                                                               |                                                                               |
|    nation_code                  gold                                          |                                                                               |
|   ======================================                                      |                                                                               |
|   'USA'                          36                                           |                                                                               |
|   'USA'                          37                                           |                                                                               |
|   'USA'                          44                                           |                                                                               |
|   'USA'                          37                                           |                                                                               |
|   'USA'                          36                                           |                                                                               |
|                                                                               |                                                                               |
| ::                                                                            |                                                                               |
|                                                                               |                                                                               |
|   $ cubrid lockdb demodb                                                      |                                                                               |
|                                                                               |                                                                               |
|   *** Lock Table Dump ***                                                     |                                                                               |
|                                                                               |                                                                               |
|   ...                                                                         |                                                                               |
|                                                                               |                                                                               |
|   Object type: Root class.                                                    |                                                                               |
|   LOCK HOLDERS:                                                               |                                                                               |
|     Tran_index =   2, Granted_mode =  IS_LOCK, Count =   1, Nsubgranules =  1 |                                                                               |
|                                                                               |                                                                               |
|   Object type: Class = participant.                                           |                                                                               |
|   LOCK HOLDERS:                                                               |                                                                               |
|     Tran_index =   2, Granted_mode =  IS_LOCK, Count =   2, Nsubgranules =  0 |                                                                               |
+-------------------------------------------------------------------------------+-------------------------------------------------------------------------------+
|                                                                               | ::                                                                            |
|                                                                               |                                                                               |
|                                                                               |   csql> UPDATE participant SET gold = 11 WHERE nation_code = 'USA';           |
+-------------------------------------------------------------------------------+-------------------------------------------------------------------------------+
| ::                                                                            |                                                                               |
|                                                                               |                                                                               |
|   csql> SELECT nation_code, gold FROM participant WHERE nation_code='USA';    |                                                                               |
|                                                                               |                                                                               |
|   /* no results until transaction 2 releases a lock */                        |                                                                               |
|                                                                               |                                                                               |
| ::                                                                            |                                                                               |
|                                                                               |                                                                               |
|   $ cubrid lockdb demodb                                                      |                                                                               |
|                                                                               |                                                                               |
|   *** Lock Table Dump ***                                                     |                                                                               |
|                                                                               |                                                                               |
|   ...                                                                         |                                                                               |
|                                                                               |                                                                               |
|   Object type: Instance of class ( 0|   551|   7) = participant.              |                                                                               |
|   LOCK HOLDERS:                                                               |                                                                               |
|       Tran_index =   3, Granted_mode =   X_LOCK, Count =   2                  |                                                                               |
|                                                                               |                                                                               |
|   ...                                                                         |                                                                               |
|                                                                               |                                                                               |
|   Object type: Root class.                                                    |                                                                               |
|   LOCK HOLDERS:                                                               |                                                                               |
|     Tran_index =   3, Granted_mode =  IX_LOCK, Count =   1, Nsubgranules =  3 |                                                                               |
|                                                                               |                                                                               |
|   NON_2PL_RELEASED:                                                           |                                                                               |
|     Tran_index =   2, Non_2_phase_lock =  IS_LOCK                             |                                                                               |
|                                                                               |                                                                               |
|   ...                                                                         |                                                                               |
|                                                                               |                                                                               |
|   Object type: Class = participant.                                           |                                                                               |
|   LOCK HOLDERS:                                                               |                                                                               |
|     Tran_index =   3, Granted_mode =  IX_LOCK, Count =   3, Nsubgranules =  5 |                                                                               |
|     Tran_index =   2, Granted_mode =  IS_LOCK, Count =   2, Nsubgranules =  0 |                                                                               |
+-------------------------------------------------------------------------------+-------------------------------------------------------------------------------+
|                                                                               | ::                                                                            |
|                                                                               |                                                                               |
|                                                                               |   csql> COMMIT;                                                               |
|                                                                               |   Execute OK. (0.000192 sec)                                                  |
+-------------------------------------------------------------------------------+-------------------------------------------------------------------------------+
| ::                                                                            |                                                                               |
|                                                                               |                                                                               |
|   nation_code                  gold                                           |                                                                               |
|   =================================                                           |                                                                               |
|   'USA'                          11                                           |                                                                               |
|   'USA'                          11                                           |                                                                               |
|   'USA'                          11                                           |                                                                               |
|   'USA'                          11                                           |                                                                               |
|   'USA'                          11                                           |                                                                               |
|                                                                               |                                                                               |
| ::                                                                            |                                                                               |
|                                                                               |                                                                               |
|   $ cubrid lockdb demodb                                                      |                                                                               |
|                                                                               |                                                                               |
|   ...                                                                         |                                                                               |
|                                                                               |                                                                               |
|   Object type: Root class.                                                    |                                                                               |
|   LOCK HOLDERS:                                                               |                                                                               |
|     Tran_index =   2, Granted_mode =  IS_LOCK, Count =   1, Nsubgranules =  1 |                                                                               |
|                                                                               |                                                                               |
|   Object type: Class = participant.                                           |                                                                               |
|   LOCK HOLDERS:                                                               |                                                                               |
|     Tran_index =   2, Granted_mode =  IS_LOCK, Count =   3, Nsubgranules =  0 |                                                                               |
|                                                                               |                                                                               |
|   ...                                                                         |                                                                               |
+-------------------------------------------------------------------------------+-------------------------------------------------------------------------------+
| ::                                                                            |                                                                               |
|                                                                               |                                                                               |
|   csql> COMMIT;                                                               |                                                                               |
|   Execute OK. (0.000192 sec)                                                  |                                                                               |
|                                                                               |                                                                               |
| ::                                                                            |                                                                               |
|                                                                               |                                                                               |
|   $ cubrid lockdb demodb                                                      |                                                                               |
|                                                                               |                                                                               |
|   ...                                                                         |                                                                               |
|                                                                               |                                                                               |
|   Object Lock Table:                                                          |                                                                               |
|           Current number of objects which are locked    = 0                   |                                                                               |
|           Maximum number of objects which can be locked = 10000               |                                                                               |
+-------------------------------------------------------------------------------+-------------------------------------------------------------------------------+

트랜잭션 교착 상태(deadlock)
----------------------------

교착 상태(deadlock)는 둘 이상의 트랜잭션이 서로 맞물려 상대방의 잠금이 해제되기를 기다리는 상태이다. 이러한 교착 상태에서는 서로가 상대방의 작업 수행을 차단하기 때문에 CUBRID는 트랜잭션 중 하나를 롤백시켜 교착 상태를 해결한다. 롤백되는 트랜잭션은 일반적으로 가장 적은 갱신을 수행한 것인데 보통 가장 최근에 시작된 트랜잭션이다. 시스템에 의해 트랜잭션이 롤백되자마자 그 트랜잭션이 가지고 있던 잠금이 해제되고 교착 상태에 있던 다른 트랜잭션이 진행되도록 허가된다.

이러한 교착 상태 발생은 예측할 수 없지만 가급적 교착 상태가 발생하지 않도록 하려면, 인덱스를 설정하여 잠금이 설정되는 범위를 줄이거나 트랜잭션을 짧게 만들거나 트랜잭션 격리 수준(isolation level)을 낮게 설정하는 것이 좋다.

에러 심각성 수준을 설정하는 시스템 파라미터인 **error_log_level** 의 값을 NOTIFICATION으로 설정하면 교착 상태 발생 시 서버 에러 로그 파일에 잠금 관련 정보가 기록된다.

다음의 에러 로그 파일 정보에서 (1)은 교착 상태를 유발한 테이블 이름을, (2)는 인덱스 이름을 나타낸다. ::

    demodb_20111102_1811.err
        ...
        OID = -532| 520| 1
    (1) Object type: Index key of class ( 0| 417| 7) = tbl.
        BTID = 0| 123| 530
    (2) Index Name : i_tbl_col1
        Total mode of holders = NS_LOCK, Total mode of waiters = NULL_LOCK.
        Num holders= 1, Num blocked-holders= 0, Num waiters= 0
        LOCK HOLDERS:
        Tran_index = 2, Granted_mode = NS_LOCK, Count = 1
    ...

**예제**

+----------------------------------------------------------------------------------------------------+----------------------------------------------------------------------------------------------------+
| session 1                                                                                          | session 2                                                                                          |
+====================================================================================================+====================================================================================================+
| ::                                                                                                 | ::                                                                                                 |
|                                                                                                    |                                                                                                    |
|   csql> ;autocommit off                                                                            |   csql> ;autocommit off                                                                            |
|                                                                                                    |                                                                                                    |
|   AUTOCOMMIT IS OFF                                                                                |   AUTOCOMMIT IS OFF                                                                                |
|                                                                                                    |                                                                                                    |
|   csql> set transaction isolation level 6;                                                         |   csql> set transaction isolation level 6;                                                         |
|                                                                                                    |                                                                                                    |
|   Isolation level set to:                                                                          |   Isolation level set to:                                                                          |
|   SERIALIZABLE                                                                                     |   SERIALIZABLE                                                                                     |
+----------------------------------------------------------------------------------------------------+----------------------------------------------------------------------------------------------------+
| ::                                                                                                 |                                                                                                    |
|                                                                                                    |                                                                                                    |
|   csql> CREATE TABLE lock_tbl(host_year integer, nation_code char(3));                             |                                                                                                    |
|   csql> INSERT INTO lock_tbl VALUES (2004, 'KOR');                                                 |                                                                                                    |
|   csql> INSERT INTO lock_tbl VALUES (2004, 'USA');                                                 |                                                                                                    |
|   csql> INSERT INTO lock_tbl VALUES (2004, 'GER');                                                 |                                                                                                    |
|   csql> INSERT INTO lock_tbl VALUES (2008, 'GER');                                                 |                                                                                                    |
|   csql> COMMIT;                                                                                    |                                                                                                    |
|                                                                                                    |                                                                                                    |
|   csql> SELECT * FROM lock_tbl;                                                                    |                                                                                                    |
|                                                                                                    |                                                                                                    |
|       host_year  nation_code                                                                       |                                                                                                    |
|   ===================================                                                              |                                                                                                    |
|            2004  'KOR'                                                                             |                                                                                                    |
|            2004  'USA'                                                                             |                                                                                                    |
|            2004  'GER'                                                                             |                                                                                                    |
|            2008  'GER'                                                                             |                                                                                                    |
+----------------------------------------------------------------------------------------------------+----------------------------------------------------------------------------------------------------+
|                                                                                                    | ::                                                                                                 |
|                                                                                                    |                                                                                                    |
|                                                                                                    |   csql> SELECT * FROM lock_tbl;                                                                    |
|                                                                                                    |                                                                                                    |
|                                                                                                    |       host_year  nation_code                                                                       |
|                                                                                                    |   ===================================                                                              |
|                                                                                                    |            2004  'KOR'                                                                             |
|                                                                                                    |            2004  'USA'                                                                             |
|                                                                                                    |            2004  'GER'                                                                             |
|                                                                                                    |            2008  'GER'                                                                             |
+----------------------------------------------------------------------------------------------------+----------------------------------------------------------------------------------------------------+
| ::                                                                                                 |                                                                                                    |
|                                                                                                    |                                                                                                    |
|   csql> DELETE FROM lock_tbl WHERE host_year=2008;                                                 |                                                                                                    |
|                                                                                                    |                                                                                                    |
|   /* no result until transaction 2 releases a lock */                                              |                                                                                                    |
|                                                                                                    |                                                                                                    |
| ::                                                                                                 |                                                                                                    |
|                                                                                                    |                                                                                                    |
|   $ cubrid lockdb demodb                                                                           |                                                                                                    |
|                                                                                                    |                                                                                                    |
|   *** Lock Table Dump ***                                                                          |                                                                                                    |
|                                                                                                    |                                                                                                    |
|   ...                                                                                              |                                                                                                    |
|                                                                                                    |                                                                                                    |
|                                                                                                    |                                                                                                    |
|   Object type: Class = lock_tbl.                                                                   |                                                                                                    |
|   LOCK HOLDERS:                                                                                    |                                                                                                    |
|       Tran_index =   2, Granted_mode =   S_LOCK, Count =   2, Nsubgranules =  0                    |                                                                                                    |
|                                                                                                    |                                                                                                    |
|   BLOCKED LOCK HOLDERS:                                                                            |                                                                                                    |
|       Tran_index =   1, Granted_mode =   S_LOCK, Count =   3, Nsubgranules =  0                    |                                                                                                    |
|       Blocked_mode = SIX_LOCK                                                                      |                                                                                                    |
|       Start_waiting_at = Fri Feb 12 14:22:58 2010                                                  |                                                                                                    |
|       Wait_for_nsecs = -1                                                                          |                                                                                                    |
+----------------------------------------------------------------------------------------------------+----------------------------------------------------------------------------------------------------+
|                                                                                                    | ::                                                                                                 |
|                                                                                                    |                                                                                                    |
|                                                                                                    |   csql> INSERT INTO lock_tbl VALUES (2004, 'AUS');                                                 |
+----------------------------------------------------------------------------------------------------+----------------------------------------------------------------------------------------------------+
| ::                                                                                                 |                                                                                                    |
|                                                                                                    |                                                                                                    |
|   ERROR: Your transaction (index 1, dba@ 090205|4760) has been unilaterally aborted by the system. |                                                                                                    |
|                                                                                                    |                                                                                                    |
|   /* System rolled back the transaction 1 to resolve a deadlock */                                 |                                                                                                    |
|                                                                                                    |                                                                                                    |
| ::                                                                                                 |                                                                                                    |
|                                                                                                    |                                                                                                    |
|   $ cubrid lockdb demodb                                                                           |                                                                                                    |
|                                                                                                    |                                                                                                    |
|   *** Lock Table Dump ***                                                                          |                                                                                                    |
|                                                                                                    |                                                                                                    |
|   Object type: Class = lock_tbl.                                                                   |                                                                                                    |
|   LOCK HOLDERS:                                                                                    |                                                                                                    |
|       Tran_index =   2, Granted_mode = SIX_LOCK, Count =   3, Nsubgranules =  0                    |                                                                                                    |
+----------------------------------------------------------------------------------------------------+----------------------------------------------------------------------------------------------------+

트랜잭션 잠금 타임아웃
----------------------

CUBRID는 트랜잭션 잠금 설정이 허용될 때까지 잠금을 대기하는 시간을 설정하는 잠금 타임아웃(lock timeout) 기능을 제공한다.

만약 설정된 잠금 타임아웃 시간 이내에 잠금이 허용되지 않으면, 잠금 타임아웃 시간이 경과된 시점에 해당 트랜잭션을 롤백시키고 에러를 출력한다. 또한, 잠금 타임아웃 시간 이내에 트랜잭션 교착 상태가 발생하면, CUBRID는 교착 상태에 있는 여러 트랜잭션 중 대기시간이 타임아웃 시간에 가까운 트랜잭션을 롤백시킨다.

**잠금 타임아웃 값 설정**

**$CUBRID/conf/cubrid.conf** 파일 내의 시스템 파라미터 **lock_timeout** 또는 **SET TRANSACTION** 구문을 통해 응용 프로그램이 잠금을 대기하는 타임아웃 시간(초 단위)을 설정하며, 설정된 시간이 경과된 이후에는 해당 트랜잭션을 롤백시키고 에러를 출력한다. **lock_timeout** 파라미터의 기본값은 **-1** 이며, 이는 트랜잭션 잠금이 허용되는 시점까지 무한정 대기한다는 의미이다. 따라서, 사용자는 응용 프로그램의 트랜잭션 패턴에 맞게 이 값을 변경할 수 있다. 만약, 잠금 타임아웃 값이 0으로 설정되면 잠금이 발생하는 즉시 에러 메시지가 출력될 것이다. ::

    SET TRANSACTION LOCK TIMEOUT timeout_spec [ ; ]
    timeout_spec:
    - INFINITE
    - OFF
    - unsigned_integer
    - variable

*   **INFINITE** : 트랜잭션 잠금이 허용될 때까지 무한정 대기한다. 시스템 파라미터 **lock_timeout**\ 을 -1로 설정한 것과 같다.
*   **OFF** : 잠금을 대기하지 않고, 해당 트랜잭션을 롤백시킨 후 에러 메시지를 출력한다. 시스템 파라미터 **lock_timeout**\ 을 0으로 설정한 것과 같다.
*   *unsigned_integer* : 초 단위로 설정되며, 설정된 시간만큼 트랜잭션 잠금을 대기한다.
*   *variable* : 변수를 지정할 수 있으며, 변수에 저장된 값만큼 트랜잭션 잠금을 대기한다.

**예제 1** ::

    vi $CUBRID/conf/cubrid.conf
    ...
    lock_timeout = 10s
    ...

**예제 2** ::

    SET TRANSACTION LOCK TIMEOUT 10;

**잠금 타임아웃 값 확인**

**GET TRANSACTION** 문을 이용하여 현재 응용 프로그램이 설정된 잠금 타임아웃 값을 확인할 수 있고, 이 값을 변수에 저장할 수도 있다. ::

    GET TRANSACTION LOCK TIMEOUT [ { INTO | TO } variable ] [ ; ]

**예제** ::

    GET TRANSACTION LOCK TIMEOUT;
    
             Result
    ===============
      1.000000e+001

**잠금 타임아웃 에러 메시지 확인과 조치 방법**

다른 트랜잭션의 잠금이 해제되기를 대기하던 트랜잭션에 대해 잠금 타임아웃이 발생하면, 아래와 같은 에러 메시지를 출력한다. ::

    Your transaction (index 2, user1@host1|9808) timed out waiting on IX_LOCK lock on class tbl. You are waiting for
    user(s) user1@host1|csql(9807), user1@host1|csql(9805) to finish.

*   Your transaction(index 2 ...) : 잠금을 대기하다가 타임아웃으로 롤백된 트랜잭션의 인덱스가 2라는 의미이다. 트랜잭션 인덱스는 클라이언트가 데이터베이스 서버에 접속하였을 때 순차적으로 할당되는 번호이다. 이는 **cubrid lockdb** 유틸리티 실행을 통해서도 확인할 수 있다.

*   (... user1\@host1|9808) : *user1* 는 클라이언트의 로그인 아이디이고, @의 뒷 부분은 클라이언트가 수행된 호스트 이름이다. 또한 | 의 뒷 부분은 클라이언트의 프로세스 ID(PID)이다.

*   IX_LOCK : 특정 행에 배타 잠금이 설정됨에 따라 상위 객체인 테이블에 의도 배타 잠금이 설정된다. 이에 관한 상세한 설명은 :ref:`lock-mode` 을 참고한다.

*   user1@host1|csql(9807), user1@host1|csql(9805) : **IX_LOCK** 잠금을 설정하기 위해 종료되기를 기다리는 다른 트랜잭션들이다.

즉, 위의 잠금 에러 메시지는 "다른 트랜잭션들이 *tbl* 테이블의 특정 행에 잠금을 점유하고 있으므로, *host1* 호스트에서 수행된 트랜잭션은 다른 트랜잭션들이 종료되기를 기다리다가 타임아웃 시간이 경과되어 롤백되었다."로 해석할 수 있다. 만약, 에러 메시지에 명시된 트랜잭션의 잠금 정보를 확인하고자 한다면, **cubrid lockdb** 유틸리티를 통해 현재 잠금을 점유 중인 클라이언트의 트랜잭션 ID 값, 클라이언트 프로그램 이름, 프로세스 ID(PID)를 확인할 수 있다. 이에 관한 상세한 설명은 :ref:`lockdb` 을 참고한다.

이처럼 트랜잭션의 잠금 정보를 확인한 후에는 SQL 로그를 통해 커밋되지 않은 질의문을 확인하여 트랜잭션을 정리할 수 있다. SQL 로그를 확인하는 방법은 :ref:`broker-logs`\ 를 참고한다.

또한, **cubrid killtran** 유틸리티를 통해 문제가 되는 트랜잭션을 강제 종료할 수 있으며, 이에 관한 상세한 설명은 :ref:`killtran` 를 참고한다.

.. _transaction-isolation-level:

트랜잭션 격리 수준
==================

트랜잭션의 격리 수준은 트랜잭션이 동시에 진행 중인 다른 트랜잭션에 의해 간섭받는 정도를 의미하며, 트랜잭션 격리 수준이 높을수록 트랜잭션 간 간섭이 적으며 직렬적이고, 트랜잭션 격리 수준이 낮을수록 트랜잭션 간 간섭은 많으나 높은 동시성을 보장한다. 사용자는 적용하고자 하는 서비스의 특성에 따라 격리 수준을 적절히 설정함으로써 데이터베이스의 일관성(consistency)과 동시성(concurrency)을 조정할 수 있다.

.. note:: 지원되는 모든 격리 수준에서 트랜잭션은 복구 가능하다. 이는 트랜잭션이 끝나기 전에는 갱신을 커밋하지 않기 때문이다.

.. _set-transaction-isolation-level:

격리 수준 설정
--------------

**$CUBRID/conf/cubrid.conf** 파일 내의 시스템 파라미터 **isolation_level** 과 **SET TRANSACTION** 문을 사용하면, 응용 프로그램에서 수행되는 트랜잭션 격리 수준을 설정할 수 있다. 기본으로 설정된 격리 수준은 **READ COMMITTED** 이며, CUBRID가 제공하는 4부터 6까지의 격리 수준 중에 4에 해당한다. 이에 관한 상세한 설명은 :ref:`database-concurrency`\ 을 참고한다. ::

    SET TRANSACTION ISOLATION LEVEL isolation_level_spec ;
    
    isolation_level_spec:
        SERIALIZABLE | 6
        REPETABLE READ | 5
        READ COMMITTED | 4

**예제 1** ::

    vi $CUBRID/conf/cubrid.conf
    ...

    isolation_level = 4
    ...

**예제 2** ::

    SET TRANSACTION ISOLATION LEVEL 4;
    -- 또는
    SET TRANSACTION ISOLATION LEVEL READ COMMITTED;

아래의 표는 CUBRID에서 지원하는 1에서 6까지의 격리 수준에 관한 설명이다. 이는 테이블 스키마와 행(row)에 대한 격리 수준 조합으로 구성된다.

**CUBRID가 지원하는 격리 수준**

+-----------------------+-----------------------------------------------------------------------------------------------------------------------------------+
| 격리 수준 이름        | 설명                                                                                                                              |
+=======================+===================================================================================================================================+
| SERIALIZABLE (6)      | 동시성 관련한 모든 문제들(더티 읽기, 반복 불가능한 읽기, 유령 읽기)이 발생하지 않는다                                             |
+-----------------------+-----------------------------------------------------------------------------------------------------------------------------------+
| REPEATABLE READ (5)   | 트랜잭션 T1이 테이블 A를 조회하는 중에 다른 트랜잭션 T2가 테이블 A의 스키마를 갱신할 수 없다.                                     |
|                       | 트랜잭션 T1이 특정 레코드를 여러 번 조회하는 중에, 다른 트랜잭션 T2가 삽입한 레코드 R에 대한 유령 읽기를 경험할 수 있다.          |
+-----------------------+-----------------------------------------------------------------------------------------------------------------------------------+
| READ COMMITTED (4)    | 트랜잭션 T1이 테이블 A를 조회하는 중에 다른 트랜잭션 T2가 테이블 A의 스키마를 갱신할 수 없다.                                     |
|                       | 트랜잭션 T1이 레코드 R을 여러 번 조회하는 중에, 다른 트랜잭션 T2가 갱신하고 커밋한 R' 읽기(반복 불가능한 읽기)를 경험할 수 있다.  |
+-----------------------+-----------------------------------------------------------------------------------------------------------------------------------+

응용 프로그램에서 트랜잭션 수행 중에 격리 수준이 변경되면, 수행 중인 트랜잭션의 남은 부분부터 변경된 격리 수준이 적용된다. 따라서, 트랜잭션 수행 중 객체에 대해 이미 획득한 일부 잠금이 새로운 격리 수준이 적용되는 동안 해제될 수도 있다. 이처럼 설정된 격리 수준이 하나의 트랜잭션 전체에 적용되는 것이 아니라 트랜잭션 중간에 변경되어 적용될 수 있기 때문에, 트랜잭션 격리 수준은 트랜잭션 시작 시점(커밋, 롤백, 또는 시스템 재시작 이후)에 변경하는 것이 하는 것이 바람직하다.

격리 수준 값 확인
-----------------

**GET TRANSACTION** 문을 이용하여 현재 클라이언트에 설정된 격리 수준 값을 출력하거나 *variable* 에 할당할 수 있다. 아래는 격리 수준을 확인하기 위한 구문이다. ::

    GET TRANSACTION ISOLATION LEVEL [ { INTO | TO } variable ] [ ; ]

.. code-block:: sql

    GET TRANSACTION ISOLATION LEVEL;
    
::

           Result
    =============
      READ COMMITTED

.. _isolation-level-6:

SERIALIZABLE
------------

가장 높은 격리 수준(6)으로서, 더티 읽기(dirty read), 반복 불가능한 읽기(non-repeatable read), 유령 읽기(phantom read) 등의 동시성 관련 문제가 발생하지 않는다.

다음과 같은 규칙이 적용된다.

*   트랜잭션 T1은 다른 트랜잭션 T2에서 갱신 중인 레코드를 읽을 수 없고, 수정할 수 없다.
*   트랜잭션 T1은 트랜잭션 T2에서 조회 중인 레코드를 읽을 수 없고, 수정할 수 없다.
*   트랜잭션 T1이 테이블 A의 레코드를 조회하는 중에 다른 트랜잭션 T2가 테이블 A로 새로운 레코드를 삽입할 수 없다.

이 격리 수준은 공유 잠금 및 배타 잠금 모두 2단계 잠금(two-phase locking) 프로토콜을 따르므로, 어떤 연산을 수행하더라도 해당 트랜잭션은 종료될 때까지 잠금을 보유한다.

**예제**

다음은 동시에 수행되는 트랜잭션의 격리 수준이 **SERIALIZABLE**\ 인 경우 한 트랜잭션에서 객체 읽기 또는 객체 갱신을 수행하는 동안 다른 트랜잭션이 테이블 또는 레코드에 접근할 수 없음을 보여주는 예제이다.

+-------------------------------------------------------------------------+----------------------------------------------------------------------------+
| session 1                                                               | session 2                                                                  |
+=========================================================================+============================================================================+
| ::                                                                      | ::                                                                         |
|                                                                         |                                                                            |
|   csql> ;autocommit off                                                 |   csql> ;autocommit off                                                    |
|                                                                         |                                                                            |
|   AUTOCOMMIT IS OFF                                                     |   AUTOCOMMIT IS OFF                                                        |
|                                                                         |                                                                            |
|   csql> SET TRANSACTION ISOLATION LEVEL 6;                              |   csql> SET TRANSACTION ISOLATION LEVEL 6;                                 |
|                                                                         |                                                                            |
|   Isolation level set to:                                               |   Isolation level set to:                                                  |
|   SERIALIZABLE                                                          |   SERIALIZABLE                                                             |
+-------------------------------------------------------------------------+----------------------------------------------------------------------------+
| ::                                                                      |                                                                            |
|                                                                         |                                                                            |
|   csql> CREATE TABLE isol6_tbl(host_year integer, nation_code char(3)); |                                                                            |
|                                                                         |                                                                            |
|   csql> INSERT INTO isol6_tbl VALUES (2008, 'AUS');                     |                                                                            |
|                                                                         |                                                                            |
|   csql> COMMIT;                                                         |                                                                            |
+-------------------------------------------------------------------------+----------------------------------------------------------------------------+
|                                                                         | ::                                                                         |
|                                                                         |                                                                            |
|                                                                         |   csql> SELECT * FROM isol6_tbl WHERE nation_code = 'AUS';                 |
|                                                                         |                                                                            |
|                                                                         |       host_year  nation_code                                               |
|                                                                         |   ===================================                                      |
|                                                                         |            2008  'AUS'                                                     |
+-------------------------------------------------------------------------+----------------------------------------------------------------------------+
| ::                                                                      |                                                                            |
|                                                                         |                                                                            |
|   csql> INSERT INTO isol6_tbl VALUES (2004, 'AUS');                     |                                                                            |
|                                                                         |                                                                            |
|   /* unable to insert a row until the tran 2 committed */               |                                                                            |
+-------------------------------------------------------------------------+----------------------------------------------------------------------------+
|                                                                         | ::                                                                         |
|                                                                         |                                                                            |
|                                                                         |   csql> COMMIT;                                                            |
+-------------------------------------------------------------------------+----------------------------------------------------------------------------+
|                                                                         | ::                                                                         |
|                                                                         |                                                                            |
|                                                                         |   csql> SELECT * FROM isol6_tbl WHERE nation_code = 'AUS';                 |
|                                                                         |                                                                            |
|                                                                         |   /* unable to select rows until tran 1 committed */                       |
+-------------------------------------------------------------------------+----------------------------------------------------------------------------+
| ::                                                                      | ::                                                                         |
|                                                                         |                                                                            |
|   csql> COMMIT;                                                         |       host_year  nation_code                                               |
|                                                                         |   ===================================                                      |
|                                                                         |            2008  'AUS'                                                     |
|                                                                         |            2004  'AUS'                                                     |
+-------------------------------------------------------------------------+----------------------------------------------------------------------------+
| ::                                                                      |                                                                            |
|                                                                         |                                                                            |
|   csql> DELETE FROM isol6_tbl                                           |                                                                            |
|   csql> WHERE nation_code = 'AUS' and                                   |                                                                            |
|   csql> host_year=2008;                                                 |                                                                            |
|                                                                         |                                                                            |
|   /* unable to delete rows until tran 2 committed */                    |                                                                            |
+-------------------------------------------------------------------------+----------------------------------------------------------------------------+
|                                                                         | ::                                                                         |
|                                                                         |                                                                            |
|                                                                         |   csql> COMMIT;                                                            |
+-------------------------------------------------------------------------+----------------------------------------------------------------------------+
|                                                                         | ::                                                                         |
|                                                                         |                                                                            |
|                                                                         |   csql> SELECT * FROM isol6_tbl WHERE nation_code = 'AUS';                 |
|                                                                         |                                                                            |
|                                                                         |   /* unable to select rows until tran 1 committed */                       |
+-------------------------------------------------------------------------+----------------------------------------------------------------------------+
| ::                                                                      | ::                                                                         |
|                                                                         |                                                                            |
|   csql> COMMIT;                                                         |       host_year  nation_code                                               |
|                                                                         |   ===================================                                      |
|                                                                         |            2004  'AUS'                                                     |
+-------------------------------------------------------------------------+----------------------------------------------------------------------------+
| ::                                                                      | ::                                                                         |
|                                                                         |                                                                            |
|   csql> ALTER TABLE isol6_tbl                                           |   /* repeatable read is ensured while tran_1 is altering table schema */   |
|                                                                         |                                                                            |
|   /* unable to alter the table schema until tran 2 committed */         |       host_year  nation_code                                               |
|                                                                         |   ===================================                                      |
|                                                                         |            2004  'AUS'                                                     |
+-------------------------------------------------------------------------+----------------------------------------------------------------------------+
|                                                                         | ::                                                                         |
|                                                                         |                                                                            |
|                                                                         |   csql> COMMIT;                                                            |
+-------------------------------------------------------------------------+----------------------------------------------------------------------------+
|                                                                         | ::                                                                         |
|                                                                         |                                                                            |
|                                                                         |   csql> SELECT * FROM isol6_tbl WHERE nation_code = 'AUS';                 |
|                                                                         |                                                                            |
|                                                                         |   /* unable to access the table until tran_1 committed */                  |
+-------------------------------------------------------------------------+----------------------------------------------------------------------------+
| ::                                                                      | ::                                                                         |
|                                                                         |                                                                            |
|   csql> COMMIT;                                                         |   host_year  nation_code  gold                                             |
|                                                                         |   ===================================                                      |
|                                                                         |     2004  'AUS'           NULL                                             |
+-------------------------------------------------------------------------+----------------------------------------------------------------------------+

.. _isolation-level-5:

REPEATABLE READ
---------------

비교적 높은 격리 수준(5)으로서, 더티 읽기, 반복 불가능한 읽기가 발생하지 않지만, 유령 읽기는 발생할 수 있다.

다음과 같은 규칙이 적용된다.

*   트랜잭션 T1은 다른 트랜잭션 T2에서 갱신 중인 레코드를 읽을 수 없고, 수정할 수 없다.
*   트랜잭션 T1은 다른 트랜잭션 T2에서 조회 중인 레코드를 읽을 수 없고, 수정할 수 없다.
*   트랜잭션 T1이 테이블 A의 레코드를 조회하는 중에 다른 트랜잭션 T2가 테이블 A로 새로운 레코드를 삽입할 수 있다. 단, 트랜잭션 T1과 T2가 동일한 레코드에 대해 잠금을 설정할 수 없다.

이 격리 수준은 2단계 잠금(two-phase locking) 프로토콜을 따른다.

**예제**

다음은 동시에 수행되는 트랜잭션의 격리 수준이 **REPEATABLE READ**\ 인 경우 한 트랜잭션에서 객체 읽기를 수행하는 동안 다른 트랜잭션이 새로운 레코드를 추가할 수 있으므로 유령 읽기가 발생할 수 있음을 보여주는 예제이다.

+----------------------------------------------------------------------------+-----------------------------------------------------------------------------+
| session 1                                                                  | session 2                                                                   |
+============================================================================+=============================================================================+
| ::                                                                         | ::                                                                          |
|                                                                            |                                                                             |
|   csql> ;autocommit off                                                    |   csql> ;autocommit off                                                     |
|                                                                            |                                                                             |
|   AUTOCOMMIT IS OFF                                                        |   AUTOCOMMIT IS OFF                                                         |
|                                                                            |                                                                             |
|   csql> SET TRANSACTION ISOLATION LEVEL 5;                                 |   csql> SET TRANSACTION ISOLATION LEVEL 5;                                  |
|                                                                            |                                                                             |
|   Isolation level set to:                                                  |   Isolation level set to:                                                   |
|   REPEATABLE READ                                                          |   REPEATABLE READ                                                           |
+----------------------------------------------------------------------------+-----------------------------------------------------------------------------+
| ::                                                                         |                                                                             |
|                                                                            |                                                                             |
|   csql> CREATE TABLE isol5_tbl(host_year integer, nation_code char(3));    |                                                                             |
|   csql> CREATE UNIQUE INDEX on isol5_tbl(nation_code, host_year);          |                                                                             |
|                                                                            |                                                                             |
|   csql> INSERT INTO isol5_tbl VALUES (2008, 'AUS');                        |                                                                             |
|   csql> INSERT INTO isol5_tbl VALUES (2004, 'AUS');                        |                                                                             |
|                                                                            |                                                                             |
|   csql> COMMIT;                                                            |                                                                             |
+----------------------------------------------------------------------------+-----------------------------------------------------------------------------+
|                                                                            | ::                                                                          |
|                                                                            |                                                                             |
|                                                                            |   csql> SELECT * FROM isol5_tbl WHERE nation_code='AUS';                    |
|                                                                            |                                                                             |
|                                                                            |       host_year  nation_code                                                |
|                                                                            |   ===================================                                       |
|                                                                            |            2004  'AUS'                                                      |
|                                                                            |            2008  'AUS'                                                      |
+----------------------------------------------------------------------------+-----------------------------------------------------------------------------+
| ::                                                                         |                                                                             |
|                                                                            |                                                                             |
|   csql> INSERT INTO isol5_tbl VALUES (2004, 'KOR');                        |                                                                             |
|   csql> INSERT INTO isol5_tbl VALUES (2000, 'AUS');                        |                                                                             |
|                                                                            |                                                                             |
|   /* able to insert new rows only when locks are not conflicted */         |                                                                             |
+----------------------------------------------------------------------------+-----------------------------------------------------------------------------+
|                                                                            | ::                                                                          |
|                                                                            |                                                                             |
|                                                                            |   csql> SELECT * FROM isol5_tbl WHERE nation_code='AUS';                    |
|                                                                            |                                                                             |
|                                                                            |   /* phantom read may occur when tran 1 committed */                        |
+----------------------------------------------------------------------------+-----------------------------------------------------------------------------+
| ::                                                                         | ::                                                                          |
|                                                                            |                                                                             |
|   csql> COMMIT;                                                            |       host_year  nation_code                                                |
|                                                                            |   ===================================                                       |
|                                                                            |            2000  'AUS'                                                      |
|                                                                            |            2004  'AUS'                                                      |
|                                                                            |            2008  'AUS'                                                      |
+----------------------------------------------------------------------------+-----------------------------------------------------------------------------+
| ::                                                                         |                                                                             |
|                                                                            |                                                                             |
|   csql> DELETE FROM isol5_tbl                                              |                                                                             |
|   csql> WHERE nation_code = 'AUS' and                                      |                                                                             |
|   csql> host_year=2008;                                                    |                                                                             |
|                                                                            |                                                                             |
|   /* unable to delete rows until tran 2 committed */                       |                                                                             |
+----------------------------------------------------------------------------+-----------------------------------------------------------------------------+
|                                                                            | ::                                                                          |
|                                                                            |                                                                             |
|                                                                            |   csql> COMMIT;                                                             |
+----------------------------------------------------------------------------+-----------------------------------------------------------------------------+
|                                                                            | ::                                                                          |
|                                                                            |                                                                             |
|                                                                            |   csql> SELECT * FROM isol5_tbl WHERE nation_code = 'AUS';                  |
|                                                                            |                                                                             |
|                                                                            |   /* unable to select rows until tran 1 committed */                        |
+----------------------------------------------------------------------------+-----------------------------------------------------------------------------+
| ::                                                                         | ::                                                                          |
|                                                                            |                                                                             |
|   csql> COMMIT;                                                            |       host_year  nation_code                                                |
|                                                                            |   ===================================                                       |
|                                                                            |            2000  'AUS'                                                      |
|                                                                            |            2004  'AUS'                                                      |
+----------------------------------------------------------------------------+-----------------------------------------------------------------------------+
| ::                                                                         |                                                                             |
|                                                                            |                                                                             |
|   csql> ALTER TABLE isol5_tbl ADD COLUMN gold INT;                         |                                                                             |
|                                                                            |                                                                             |
|   /* unable to alter the table schema until tran 2 committed */            |                                                                             |
+----------------------------------------------------------------------------+-----------------------------------------------------------------------------+
|                                                                            | ::                                                                          |
|                                                                            |                                                                             |
|                                                                            |   /* repeatable read is ensured while tran_1 is altering table schema */    |
|                                                                            |                                                                             |
|                                                                            |   csql> SELECT * FROM isol5_tbl WHERE nation_code = 'AUS';                  |
|                                                                            |                                                                             |
|                                                                            |       host_year  nation_code                                                |
|                                                                            |   ===================================                                       |
|                                                                            |            2000  'AUS'                                                      |
|                                                                            |            2004  'AUS'                                                      |
+----------------------------------------------------------------------------+-----------------------------------------------------------------------------+
|                                                                            | ::                                                                          |
|                                                                            |                                                                             |
|                                                                            |   csql> COMMIT;                                                             |
+----------------------------------------------------------------------------+-----------------------------------------------------------------------------+
|                                                                            | ::                                                                          |
|                                                                            |                                                                             |
|                                                                            |   csql> SELECT * FROM isol5_tbl WHERE nation_code = 'AUS';                  |
|                                                                            |                                                                             |
|                                                                            |   /* unable to access the table until tran_1 committed */                   |
+----------------------------------------------------------------------------+-----------------------------------------------------------------------------+
| ::                                                                         | ::                                                                          |
|                                                                            |                                                                             |
|   csql> COMMIT;                                                            |   host_year  nation_code  gold                                              |
|                                                                            |   ===================================                                       |
|                                                                            |     2000  'AUS'           NULL                                              |
|                                                                            |     2004  'AUS'           NULL                                              |
+----------------------------------------------------------------------------+-----------------------------------------------------------------------------+

.. _isolation-level-4:

READ COMMITTED
--------------

비교적 낮은 격리 수준(4)으로서 더티 읽기는 발생하지 않지만, 반복 불가능한 읽기와 유령 읽기는 발생할 수 있다. 즉, 트랜잭션 T1이 하나의 객체를 반복하여 조회하는 동안 다른 트랜잭션 T2에서의 삽입 또는 갱신이 허용되어, 트랜잭션 T1이 다른 값을 읽을 수 있다는 의미이다.

다음과 같은 규칙이 적용된다.

*   트랜잭션 T1은 다른 트랜잭션 T2에서 갱신 중인 레코드를 읽을 수 없다.
*   트랜잭션 T1은 다른 트랜잭션 T2에서 조회 중인 테이블에 대해 레코드를 갱신/삽입할 수 있다.
*   트랜잭션 T1은 다른 트랜잭션 T2에서 조회 중인 테이블의 스키마를 변경할 수 없다.

이 격리 수준은 배타 잠금에 대해서는 2단계 잠금(two-phase locking)을 따른다. 하지만 행에 대한 공유 잠금은 행이 조회된 직후 바로 해제되지만, 테이블에 대한 의도 잠금은 스키마에 대한 반복 가능한 읽기를 보장하기 위하여 트랜잭션이 종료될 때 해제된다. 

**예제**

다음은 동시에 수행되는 트랜잭션의 격리 수준이 **READ COMMITTED**\ 인 경우 한 트랜잭션에서 객체 읽기를 수행하는 동안 다른 트랜잭션이 새로운 레코드를 추가 또는 갱신할 수 있으므로 유령 읽기 및 반복 불가능한 읽기가 발생할 수 있으나, 테이블 스키마 갱신에 대해서는 반복 가능한 읽기를 보장함을 보여주는 예제이다.

+-------------------------------------------------------------------------+----------------------------------------------------------------------------------+
| session 1                                                               | session 2                                                                        |
+=========================================================================+==================================================================================+
| ::                                                                      | ::                                                                               |
|                                                                         |                                                                                  |
|   csql> ;autocommit off                                                 |   csql> ;autocommit off                                                          |
|                                                                         |                                                                                  |
|   AUTOCOMMIT IS OFF                                                     |   AUTOCOMMIT IS OFF                                                              |
|                                                                         |                                                                                  |
|   csql> SET TRANSACTION ISOLATION LEVEL 4;                              |   csql> SET TRANSACTION ISOLATION LEVEL 4;                                       |
|                                                                         |                                                                                  |
|   Isolation level set to:                                               |   Isolation level set to:                                                        |
|   READ COMMITTED                                                        |   READ COMMITTED                                                                 |
+-------------------------------------------------------------------------+----------------------------------------------------------------------------------+
| ::                                                                      |                                                                                  |
|                                                                         |                                                                                  |
|   csql> CREATE TABLE isol4_tbl(host_year integer, nation_code char(3)); |                                                                                  |
|                                                                         |                                                                                  |
|   csql> INSERT INTO isol4_tbl VALUES (2008, 'AUS');                     |                                                                                  |
|                                                                         |                                                                                  |
|   csql> COMMIT;                                                         |                                                                                  |
+-------------------------------------------------------------------------+----------------------------------------------------------------------------------+
|                                                                         | ::                                                                               |
|                                                                         |                                                                                  |
|                                                                         |   csql> SELECT * FROM isol4_tbl;                                                 |
|                                                                         |                                                                                  |
|                                                                         |       host_year  nation_code                                                     |
|                                                                         |   ===================================                                            |
|                                                                         |            2008  'AUS'                                                           |
+-------------------------------------------------------------------------+----------------------------------------------------------------------------------+
| ::                                                                      |                                                                                  |
|                                                                         |                                                                                  |
|   csql> INSERT INTO isol4_tbl VALUES (2004, 'AUS');                     |                                                                                  |
|   csql> INSERT INTO isol4_tbl VALUES (2000, 'NED');                     |                                                                                  |
|                                                                         |                                                                                  |
|   /* able to insert new rows even if tran 2 uncommitted */              |                                                                                  |
+-------------------------------------------------------------------------+----------------------------------------------------------------------------------+
|                                                                         | ::                                                                               |
|                                                                         |                                                                                  |
|                                                                         |   csql> SELECT * FROM isol4_tbl;                                                 |
|                                                                         |                                                                                  |
|                                                                         |   /* phantom read may occur when tran 1 committed */                             |
+-------------------------------------------------------------------------+----------------------------------------------------------------------------------+
| ::                                                                      | ::                                                                               |
|                                                                         |                                                                                  |
|   csql> COMMIT;                                                         |       host_year  nation_code                                                     |
|                                                                         |   ===================================                                            |
|                                                                         |            2008  'AUS'                                                           |
|                                                                         |            2004  'AUS'                                                           |
|                                                                         |            2000  'NED'                                                           |
+-------------------------------------------------------------------------+----------------------------------------------------------------------------------+
| ::                                                                      |                                                                                  |
|                                                                         |                                                                                  |
|   csql> INSERT INTO isol4_tbl VALUES (1994, 'FRA');                     |                                                                                  |
+-------------------------------------------------------------------------+----------------------------------------------------------------------------------+
|                                                                         | ::                                                                               |
|                                                                         |                                                                                  |
|                                                                         |   csql> SELECT * FROM isol4_tbl;                                                 |
|                                                                         |                                                                                  |
|                                                                         |   /* unrepeatable read may occur when tran 1 committed */                        |
+-------------------------------------------------------------------------+----------------------------------------------------------------------------------+
| ::                                                                      |                                                                                  |
|                                                                         |                                                                                  |
|   csql> DELETE FROM isol4_tbl                                           |                                                                                  |
|   csql> WHERE nation_code = 'AUS' and                                   |                                                                                  |
|   csql> host_year=2008;                                                 |                                                                                  |
|                                                                         |                                                                                  |
|   /* able to delete rows while tran 2 is selecting rows*/               |                                                                                  |
+-------------------------------------------------------------------------+----------------------------------------------------------------------------------+
| ::                                                                      | ::                                                                               |
|                                                                         |                                                                                  |
|   csql> COMMIT;                                                         |       host_year  nation_code                                                     |
|                                                                         |   ===================================                                            |
|                                                                         |            2004  'AUS'                                                           |
|                                                                         |            2000  'NED'                                                           |
|                                                                         |            1994  'FRA'                                                           |
+-------------------------------------------------------------------------+----------------------------------------------------------------------------------+
| ::                                                                      |                                                                                  |
|                                                                         |                                                                                  |
|   csql> ALTER TABLE isol4_tbl ADD COLUMN gold INT;                      |                                                                                  |
|                                                                         |                                                                                  |
|   /* unable to alter the table schema until tran 2 committed */         |                                                                                  |
+-------------------------------------------------------------------------+----------------------------------------------------------------------------------+
|                                                                         | ::                                                                               |
|                                                                         |                                                                                  |
|                                                                         |   /* repeatable read is ensured while tran_1 is altering table schema */         |
|                                                                         |                                                                                  |
|                                                                         |   csql> SELECT * FROM isol4_tbl;                                                 |
|                                                                         |                                                                                  |
|                                                                         |       host_year  nation_code                                                     |
|                                                                         |   ===================================                                            |
|                                                                         |            2004  'AUS'                                                           |
|                                                                         |            2000  'NED'                                                           |
|                                                                         |            1994  'FRA'                                                           |
+-------------------------------------------------------------------------+----------------------------------------------------------------------------------+
|                                                                         | ::                                                                               |
|                                                                         |                                                                                  |
|                                                                         |   csql> COMMIT;                                                                  |
+-------------------------------------------------------------------------+----------------------------------------------------------------------------------+
|                                                                         | ::                                                                               |
|                                                                         |                                                                                  |
|                                                                         |   csql> SELECT * FROM isol4_tbl;                                                 |
|                                                                         |                                                                                  |
|                                                                         |   /* unable to access the table until tran_1 committed */                        |
+-------------------------------------------------------------------------+----------------------------------------------------------------------------------+
| ::                                                                      | ::                                                                               |
|                                                                         |                                                                                  |
|   csql> COMMIT;                                                         |   host_year  nation_code  gold                                                   |
|                                                                         |   ===================================                                            |
|                                                                         |     2004  'AUS'           NULL                                                   |
|                                                                         |     2000  'NED'           NULL                                                   |
|                                                                         |     1994  'FRA'           NULL                                                   |
+-------------------------------------------------------------------------+----------------------------------------------------------------------------------+


.. _dirty-record-flush:

CUBRID에서 더티 레코드를 다루는 방법
------------------------------------

CUBRID는 다양한 상황에서 클라이언트의 버퍼에 존재하는 더티 데이터(또는 더티 레코드)를 데이터베이스 서버로 내려쓰기(flush)한다. 아래에 명시된 상황이 아닌 경우에도 내려쓰기가 발생할 수 있다.

*   트랜잭션 커밋이 수행될 때 더티 데이터는 서버로 내려쓰기된다.
*   클라이언트의 버퍼에 적재된 데이터가 많은 경우, 일부 더티 데이터는 서버로 내려쓰기된다.
*   테이블 *A* 의 더티 데이터는 테이블 *A* 의 스키마가 갱신될 때 서버로 내려쓰기된다.
*   테이블 *A* 의 더티 데이터는 테이블 *A* 가 조회( **SELECT** )될 때 서버로 내려쓰기된다.
*   더티 데이터의 일부는 서버 함수가 호출될 때 내려쓰기될 수 있다.

트랜잭션 종료와 복구
====================

CUBRID에서 복구 프로세스를 사용하면 소프트웨어 또는 하드웨어에 오류가 발생하더라도 데이터베이스에는 영향을 미치지 않도록 할 수 있다. CUBRID에서 모든 읽기와 갱신 명령문은 원자성(atomic)을 보장한다. 이것은 명령문들이 커밋되어 데이터베이스가 갱신되거나, 커밋되지 않아 갱신이 무효화되어야 함을 의미한다. 원자성의 개념은 트랜잭션을 구성하는 연산의 집합으로 확장된다. 트랜잭션은 커밋을 성공하여 모든 영향이 데이터베이스에 영구화 되거나 아니면 롤백되어 트랜잭션의 모든 영향이 제거되어야 한다. 트랜잭션의 원자성을 보장하기 위해서 CUBRID는 모든 트랜잭션의 갱신이 디스크에 쓰여지지 않은 채 오류가 발생할 때마다 커밋된 트랜잭션의 영향을 다시 적용시킨다. 또한 CUBRID는 사이트가 실패(몇몇 트랜잭션이 커밋되지 못했거나 응용 프로그램이 트랜잭션 취소를 요청했을 수 있다)할 때마다 데이터베이스에서 부분적으로 커밋된 트랜잭션의 영향을 제거한다. 이러한 복구 기능은 응용 프로그램이 시스템 오류에 따라 어떻게 데이터베이스를 일관성 있는 상태로 되돌릴 지에 대한 부담을 덜어준다. CUBRID에서 사용되는 복구 기능은 언두/리두 로깅 기법을 기반으로 한다.

CUBRID는 하드웨어와 소프트웨어 오류가 발생하는 동안 트랜잭션의 원자성을 유지하기 위해서 자동 복구 기법을 제공한다. CUBRID의 복구 기능은 응용 프로그램 또는 컴퓨터 시스템의 오류가 발생하더라도 데이터베이스를 항상 일관된 상태로 되돌려놓기 때문에 사용자는 복구에 대한 책임을 가질 필요가 없다. 이것을 위해 CUBRID는 시스템이나 응용 프로그램의 실패 또는 사용자의 명시적 요청에 따라 커밋된 트랜잭션의 일부를 자동적으로 롤백한다. 예를 들어 **COMMIT WORK** 문이 수행되는 동안 발생한 시스템 오류는 트랜잭션이 아직 커밋되지 않았다면(사용자의 연산이 커밋되었다는 확인을 받지 못한다) 중단해야 할 것이다. 자동 중단은 커밋되지 않은 갱신을 취소함으로써 데이터베이스에 원하지 않은 변경을 야기하는 오류를 방지한다.

데이터베이스 재구동
-------------------

CUBRID는 시스템과 매체(디스크)에 오류가 발생했을 때 커밋되었거나 커밋되지 않은 트랜잭션을 복구하기 위해 로그 볼륨/파일과 데이터베이스 백업을 이용한다. 로그는 사용자가 지정한 롤백을 지원하는데도 사용된다. 로그는 CUBRID가 생성한 순차적인 파일의 모음으로 구성된다. 가장 최근의 로그를 활성 로그(active log)라고 부르며, 나머지 로그를 보관 로그(archive log)라고 부른다. 로그 파일은 활성 로그와 보관 로그 전체를 가리키는데 사용된다.

데이터베이스에 대한 모든 갱신은 로그에 기록된다. 실제로 갱신에 대한 2개의 복사본이 기록되는데, 첫 번째 복사본은 before 이미지(UNDO log)라고 불리며 사용자가 명시한 **ROLLBACK WORK** 문이 수행되는 동안이나 매체 또는 시스템에 오류가 발생했을 때 데이터를 복원하는데 사용된다. 두 번째 복사본은 after 이미지(REDO log)인데 매체 또는 시스템에 오류가 발생했을 때 갱신을 다시 적용시키는데 사용된다.

CUBRID는 활성 로그가 꽉 차면 보관 로그로 복사하여 디스크에 보존한다. 보관 로그는 시스템 장애가 발생했을 때 데이터베이스 복구를 위해 필요하다.

**정상적인 종료 또는 오류**

데이터베이스가 정상적인 종료나 장비의 오류로 다시 시작되면 CUBRID는 자동적으로 데이터베이스를 복구한다. 복구 프로세스는 데이터베이스에 빠져있는 커밋된 변화를 다시 적용하고 데이터베이스에 저장되어 있는 커밋되지 않은 변경을 제거한다. 데이터베이스 시스템의 일반적인 연산은 복구가 끝나고 난 후 재개된다. 이러한 복구 프로세스는 어떠한 보관 로그나 데이터베이스 백업도 사용하지 않는다.

데이터베이스는 **cubrid server** 유틸리티를 이용해 재구동할 수 있다.

**매체 오류**

매체에 오류가 발생 한 후에 데이터베이스를 다시 구동시키는 데는 사용자의 개입이 다소 필요하다. 첫 번째 단계는 좋은 상태로 알려진 데이터베이스의 백업을 설치하여 데이터베이스를 복원하는 것이다. CUBRID에서 가장 최근의 로그 파일(마지막 백업 이후의 것)을 설치하는 것을 필요로 한다. 이 특정 로그(보관, 활성)는 데이터베이스의 백업 복사본에 적용된다. 복원이 커밋된 후 데이터베이스는 일반적인 종료의 경우와 마찬가지로 재구동할 수 있다.

.. note::

    데이터베이스의 정보를 잃어버릴 가능성을 줄이기 위해서 보관 로그가 디스크에서 삭제되기 전에 보관 로그의 스냅샷을 만들고 이를 백업 장치에 보관할 것을 권장한다. DBA는 **cubrid backupdb**, **cubrid restoredb** 유틸리티를 사용하여 데이터베이스를 백업하고 복원할 수 있다. 이 유틸리티에 대한 상세한 내용을 보려면 :ref:`backupdb`\를 참고한다.
